# Miten WebXR toimii Servossa

## Terminologia

Servon WebXR-toteutus koostuu kolmesta pääkomponentista:

1. Script-säie (ajaa kaiken JS:n sivulle)
2. WebGL-säie (ylläpitää WebGL-canvas-datan ja kutsuu GL-operaatioita, jotka vastaavat [WebGL API:ja](https://registry.khronos.org/webgl/specs/latest/1.0/))
3. Compositor (eli pääsäie)

Lisäksi on useita WebXR-spesifisiä käsitteitä:

* [discovery object](https://doc.servo.org/webxr_api/trait.DiscoveryAPI.html) (eli miten Servo havaitsee, voiko laite tarjota WebXR-session)
* [WebXR registry](https://doc.servo.org/webxr_api/struct.MainThreadRegistry.html) (compositorin rajapinta WebXR:ään)
* [layer manager](https://doc.servo.org/webxr_api/layer/trait.LayerManagerAPI.html) (hallinnoi WebXR-kerroksia tietylle sessiolle ja kehystoimintoja näillä kerroksilla)
* [layer grand manager](https://doc.servo.org/webxr_api/layer/trait.LayerGrandManagerAPI.html) (hallinnoi kaikkia layer managereita WebXR-sessioille)

Lopuksi on grafiikkaan liittyviä käsitteitä, jotka ovat tärkeitä WebXR-renderöinnin matalan tason yksityiskohtien kannalta:

* [Surfman](https://github.com/servo/servo/blob/8683f97fcc75b3ce12d9068e8b16b1da7e6abe78/components/webxr/glwindow/mod.rs#L466-L470) on crate, joka abstrahoi alustakohtaiset yksityiskohdat OpenGL-laitteistokiihdytetystä renderöinnistä.
* [surface](https://doc.servo.org/surfman/platform/unix/default/surface/type.Surface.html) on laitteistopuskuri, joka on sidottu tiettyyn OpenGL-kontekstiin.
* [surface texture](https://doc.servo.org/surfman/platform/unix/default/surface/type.SurfaceTexture.html) on OpenGL-tekstuuri, joka wrap:aa pinnan.
  Surface textureja voidaan jakaa OpenGL-kontekstien välillä.
* [surfman context](https://doc.servo.org/surfman/platform/unix/default/context/type.Context.html) edustaa tiettyä OpenGL-kontekstia, ja sen taustalla on alustakohtaisia toteutuksia (kuten EGL Unix-pohjaisilla alustoilla).
* [ANGLE](https://github.com/servo/mozangle/) on OpenGL-toteutus Direct3D:n päällä, jota Servo käyttää tarjotakseen yhtenäisen OpenGL-backendin Windows-alustoilla.

## Miten Servon compositor käynnistyy

Upottaja on vastuussa ikkunan luomisesta ja [renderöintikontekstin luonnin käynnistämisestä](https://github.com/servo/servo/blob/8683f97fcc75b3ce12d9068e8b16b1da7e6abe78/ports/servoshell/desktop/headed_window.rs#L147) asianmukaisesti.
Servo luo renderöintikontekstin [luomalla surfman-kontekstin](https://github.com/servo/servo/blob/8683f97fcc75b3ce12d9068e8b16b1da7e6abe78/components/shared/compositing/rendering_context.rs#L108-L120), jota [compositor käyttää](https://github.com/servo/servo/blob/8683f97fcc75b3ce12d9068e8b16b1da7e6abe78/components/servo/lib.rs#L479) kaikkiin [web-sisällön renderöintitoimintoihin](https://github.com/servo/servo/blob/8683f97fcc75b3ce12d9068e8b16b1da7e6abe78/components/servo/lib.rs#L278-L285).

## Miten sessio käynnistyy

Kun verkkosivu kutsuu `navigator.xr.requestSession(..)` JS:n kautta, tämä vastaa Servon [XrSystem::RequestSession](https://github.com/servo/servo/blob/8683f97fcc75b3ce12d9068e8b16b1da7e6abe78/components/script/dom/webxr/xrsystem.rs#L158) -metodia.
Tämä metodi [lähettää viestin](https://github.com/servo/servo/blob/3f6e2679dcc3c142750e17b4e152e4e0660542a2/components/shared/webxr/registry.rs#L80-L85) [WebXR-viestinkäsittelijälle](https://github.com/servo/servo/blob/3f6e2679dcc3c142750e17b4e152e4e0660542a2/components/shared/webxr/registry.rs#L170-L172), joka elää pääsäieellä [compositorin hallinnassa](https://github.com/servo/servo/blob/3f6e2679dcc3c142750e17b4e152e4e0660542a2/components/compositing/compositor.rs#L1420).

WebXR-viestinkäsittelijä käy läpi kaikki tunnetut discovery objectit ja yrittää [pyytää sessiota](https://github.com/servo/servo/blob/3f6e2679dcc3c142750e17b4e152e4e0660542a2/components/shared/webxr/registry.rs#L194-L209) jokaiselta.
Discovery objectit kapseloivat session luomisen jokaiselle tuetulle backendille.

Heinäkuun 19, 2024 tilanteessa on kolme WebXR-backendiä:

* [headless](https://github.com/servo/servo/tree/main/components/webxr/headless) - tukee ikkunatonta, laitteetonta laitetta automatisoiduille testeille
* [glwindow](https://github.com/servo/servo/tree/main/components/webxr/glwindow) - tukee GL-pohjaista ikkunaa manuaaliseen testaukseen työpöytäympäristöissä ilman oikeita laitteita
* [openxr](https://github.com/servo/servo/tree/main/components/webxr/openxr) - tukee laitteita, jotka toteuttavat OpenXR-standardin

WebXR-sessioiden täytyy [luoda layer manager](https://github.com/servo/servo/blob/8683f97fcc75b3ce12d9068e8b16b1da7e6abe78/components/webxr/glwindow/mod.rs#L466-L470)
jossain vaiheessa, jotta ne voivat luoda ja renderöidä WebXR-kerroksiin.
Tämä tapahtuu useassa vaiheessa:

1. Jotakin alustusta tapahtuu pääsäieellä
2. Pääsäie [lähettää synkronisen viestin](https://github.com/servo/servo/blob/3f6e2679dcc3c142750e17b4e152e4e0660542a2/components/webgl/webxr.rs#L186-L191) WebGL-säieelle
3. WebGL-säie [vastaanottaa viestin](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webgl/webgl_thread.rs#L415-L420)
4. Backend-spesifistä, grafiikka-spesifistä alustusta tapahtuu WebGL-säieellä, [layer manager factory](https://doc.servo.org/webxr_api/struct.LayerManagerFactory.html) -abstraktion takana
5. Uusi layer manager [tallennetaan WebGL-säieelle](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webgl/webxr.rs#L58-L64)
6. Pääsäie [vastaanottaa yksilöllisen tunnisteen](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webgl/webxr.rs#L193-L201) uudelle layer managerille

Tämä säieiden välinen tanssi on tärkeä, koska renderöintiä suorittavalla laitteella on usein tiukat vaatimukset yhteensopivuudelle kaikille renderöintiin käytetyille WebGL-konteksteille, ja suurin osa GL-tilasta on havaittavissa vain säieellä, joka loi sen.

## Miten OpenXR-sessio luodaan

OpenXR-discovery-prosessi alkaa [OpenXrDiscovery::request_session](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webxr/openxr/mod.rs#L277) -kohdasta.
Discovery objectilla on pääsy vain siihen tilaan, joka välitettiin sen konstruktorissa, sekä [SessionBuilder](https://doc.servo.org/webxr_api/struct.SessionBuilder.html) -objektiin, joka sisältää uuden session luomiseen tarvittavat arvot.

OpenXR-session luominen [luo ensin OpenXR-instanssin](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webxr/openxr/mod.rs#L193), joka mahdollistaa käytössä olevien laajennusten konfiguroinnin.
Eri alustoilla OpenXR:n alustukseen käytetään eri laajennuksia; Windowsilla käytetään [XR_KHR_D3D11_enable extension](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webxr/openxr/graphics_d3d11.rs#L31), koska Servo luottaa ANGLEen OpenGL-toteutuksessaan.

Kun OpenXR-instanssi on olemassa, session builderia käytetään uuden WebXR-session luomiseen, joka [ajaa omassa säieessään](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webxr/openxr/mod.rs#L305).
Kaikki WebXR-sessiot voivat joko [ajaa säieessä](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/shared/webxr/session.rs#L467-L492) tai Servo voi [ajaa ne pääsäieellä](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/shared/webxr/session.rs#L494-L507).
Tällä valinnalla on vaikutuksia siihen, miten WebXR-session grafiikka voidaan konfiguroida, sen perusteella, mitä GL-tilaa täytyy olla saatavilla jaettavaksi.

OpenXR:n uusi session-säie [alustaa OpenXR-laitteen](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webxr/openxr/mod.rs#L306-L311), joka on vastuussa varsinaisen OpenXR-session luomisesta.
Tämä session-objekti [luodaan WebGL-säieellä](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webxr/openxr/mod.rs#L847-L877) osana OpenXR layer managerin luomista, koska se luottaa [WebGL-säieen käyttämän GPU-laitteen jakamiseen](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webxr/openxr/graphics_d3d11.rs#L58-L80).

Kun session-objekti on luotu, pääsäie voi [hankkia kopion](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webxr/openxr/mod.rs#L877) ja jatkaa uuden laitteen [jäljellä olevien ominaisuuksien alustusta](https://github.com/servo/servo/blob/442eec2d5fed10572ea8f5f3dccfa49218988e5e/components/webxr/openxr/mod.rs#L881-L1032).
