# Oppaat

Tämä osio sisältää muutamia oppaita yleisiin tehtäviin Servon parissa työskennellessä.

# Verkkosisällön bugien korjaaminen

Servossa havaittavissa on pääasiassa kahta web-yhteensopivuusongelman luokkaa.
Visuaaliset bugit johtuvat usein puuttuvista ominaisuuksista tai bugeista Servon CSS- ja layout-tuessa,
kun taas interaktiivisuusongelmat ja rikkinäinen sisältö johtuvat usein bugeista tai puuttuvista ominaisuuksista
Servon DOM- ja JavaScript-toteutuksessa.

- Apua DOM-toteutuksen bugin korjaamiseen: [DOM-virheiden diagnosointi](diagnosing-dom-errors.md).
- Apua ongelmien kaventamiseen, olipa kyse layoutista tai DOM:sta: [Minimaalinen toistettava testitapaus](minimal-reproducible-test-cases.md).

# Uusien ominaisuuksien lisääminen

Ominaisuuksien lisääminen ei ole hyvä tehtävä uudelle kontribuuttorille, koska web-moottorissa ominaisuuden täydellinen toteuttaminen vaatii usein pitkän sarjan muutoksia.
Aloita lukemalla [DOM-rajapinnan toteuttaminen](implementing-a-dom-api.md).
