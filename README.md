[![FIWARE Banner](https://fiware.github.io/tutorials.Identity-Management/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Security](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/security.svg)](https://github.com/FIWARE/catalogue/blob/master/security/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Identity-Management.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)

This tutorial is an introduction to [FIWARE Keyrock](https://fiware-idm.readthedocs.io/en/latest/) - a generic enabler
which introduces **Identity Management** into FIWARE services. The tutorial explains how to create users and
organizations in preparation to assign roles and permissions to them in a later tutorial.

The tutorial demonstrates examples of interactions using the **Keyrock** GUI, as well [cUrl](https://ec.haxx.se/)
commands used to access the **Keyrock** REST API -
[Postman documentation](https://www.postman.com/downloads/) is also available.

# Start-Up

## NGSI-v2 Smart Supermarket

**NGSI-v2** offers JSON based interoperability used in individual Smart Systems. To run this tutorial with **NGSI-v2**, use the `NGSI-v2` branch.

```console
git clone https://github.com/FIWARE/tutorials.Identity-Management.git
cd tutorials.Identity-Management
git checkout NGSI-v2

./services create
./services start
```

| [![NGSI v2](https://img.shields.io/badge/NGSI-v2-5dc0cf.svg)](https://fiware-ges.github.io/orion/api/v2/stable/) | :books: [Documentation](https://github.com/FIWARE/tutorials.Identity-Management/tree/NGSI-v2) | <img src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.Identity-Management/) |  ![](https://img.shields.io/github/last-commit/fiware/tutorials.Identity-Management/NGSI-v2) 
| --- | --- | --- | ---


## NGSI-LD Smart Farm

**NGSI-LD** offers JSON-LD based interoperability used for Federations and Data Spaces. To run this tutorial with **NGSI-LD**, use the `NGSI-LD` branch.

```console
git clone https://github.com/FIWARE/tutorials.Identity-Management.git
cd tutorials.Identity-Management
git checkout NGSI-LD

./services create
./services start
```

| [![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://cim.etsi.org/NGSI-LD/official/front-page.html) | :books: [Documentation](https://github.com/FIWARE/tutorials.Identity-Management/tree/NGSI-LD) | <img  src="https://cdn.jsdelivr.net/npm/simple-icons@v3/icons/postman.svg" height="15" width="15"> [Postman Collection](https://fiware.github.io/tutorials.Identity-Management/ngsi-ld.html) |  ![](https://img.shields.io/github/last-commit/fiware/tutorials.Identity-Management/NGSI-LD)
| --- | --- | --- | ---


---

## License

[MIT](LICENSE) © 2018-2024 FIWARE Foundation e.V.
