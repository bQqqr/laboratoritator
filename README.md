<a name="readme-top"></a>

<!-- PROJECT SHIELDS -->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]



<!-- PROJECT LOGO -->
<br />
<div align="center">


  <a href="https://github.com/bQqqr/laboratoritator">
    <img src=".github/logo.jpg" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">Laboratoritator</h3>

  <p align="center">
    🏭 How to setup your organization's internal services with a local CA. 
    <br />
    <br />
    <br />
    <a href="https://traefik.moxthos.art">View Demo</a>
    ·
    <a href="https://github.com/bQqqr/laboratoritator/issues">Report Bug</a>
    ·
    <a href="https://github.com/bQqqr/laboratoritator/issues">Request Feature</a>
  </p>
</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
    </li>
    <li>
      <a href="#roadmap">Roadmap</a>
    </li>
    <li>
      <a href="#contributing">Contributing</a>
    </li>
    <li>
      <a href="#license">License</a>
    </li>
    <li>
      <a href="#contact">Contact</a>
    </li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

For various reasons, many development teams do not trust cloud providers and want to host their own in-house services. Laboratoriator is a series of docker-compose files and an instructional guide on how to setup your organization's services in a internal network.

- https://traefik.moxthos.art
- https://gitea.moxthos.art
- https://wikijs.moxthos.art
- https://nextcloud.moxthos.art
- https://rocketchat.moxthos.art

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->
## Getting Started

### Prerequirements

Install the following inside your host.

- [docker][docker-installation-url]
- [docker compose plugin][docker-compose-installation-url]
- [lazydocker][lazydocker-installation-url]
- [step][step-installation-url]

### Install Step CA

Create the Step's directory and modify it, so docker can write in it.

```
$ cd $HOME
$ mkdir stepca
$ chown 1000:1000 stepca
```
Then, initialize and configurate Step. Make sure that you save the password and the new CA's fingerprint.

```
$ docker run -p 8443:8443 -it -v `pwd`/stepca:/home/step smallstep/step-ca:0.18.1 step ca init
> Standalone
> MoxthosCA
> 0.0.0.0,ca.moxthos.art
> :8443
> admin@moxthos.art
> ********
...
✔ Root fingerprint: ******************************************************
...
```

Copy laboratoritator's `containers/stepca.docker-compose.yml` to your host. Save your password inside the container and finally run it.

```
$ docker run -p 8443:8443 -it -v `pwd`/stepca:/home/step smallstep/step-ca:0.18.1 sh
(inside container) $ echo -n '**********' > /home/step/secrets/password
(inside container) $ exit
$ docker compose -f stepca.docker-compose.yml up -d
```

Do not forget to change the default max lifetime for TLS certificates. Open `~/stepca/config/ca.json` and configure `claims` inside the `authority` or the `provisioner` object. Then, restart the container to apply the changes.

```
"claims": {
    "maxTLSCertDuration": "2160h",
    "defaultTLSCertDuration": "24h",
}
```

Install the root certificate in your host using `step`.

```
$ step ca bootstrap --ca-url https://ca.moxthos.art:8443 --fingerprint ************ --install
```

### Install Traefik

Firstly, create `certs` and `traefik` directories. Also modify them so docker can write in them.

```
$ mkdir certs traefik
$ chown 1000:1000 certs traefik
```

Create a certificate/key for Traefik and save it inside `certs`.

```
$ step ca certificate --ca-url https://ca.moxthos.art:8443 traefik.moxthos.art certs/traefik.crt certs/traefik.key --not-after 2399h
```

Copy the root certificate from Step's container to `./certs`.

```
$ docker cp stepca:/home/step/certs/root_ca.crt ./certs/
$ chmod 644 ./certs/root_ca.crt
```

Create a file named `traefik-config.toml` inside `~/traefik/`. Make sure that it contains the following:

```
[[tls.certificates]]
  certFile = "/certs/traefik.crt"
  keyFile = "/certs/traefik.key"
```

Copy laboratoriator's `containers/traefik.docker-compose.yml` to your home directory. Run it.

```
$ docker compose -f traefik.docker-compose.yml up -d
```

### Install Gitea

Create a certificate/key for Gitea and save it inside `certs`.

```
$ step ca certificate --ca-url https://ca.moxthos.art:8443 gitea.moxthos.art certs/gitea.crt certs/gitea.key --not-after 2399h
```

Make sure that `~/traefik/traefik-config.toml` contains the following:
```
[[tls.certificates]]
  certFile = "/certs/gitea.crt"
  keyFile = "/certs/gitea.key"
```

Copy laboratoriator's `containers/gitea.docker-compose.yml` to your host's home directory and run it.

```
$ docker compose -f gitea.docker-compose.yml up -d
```

### Install WikiJS

Create a certificate/key for WikiJS and save it inside `certs`.

```
$ step ca certificate --ca-url https://ca.moxthos.art:8443 wikijs.moxthos.art certs/wikijs.crt certs/wikijs.key --not-after 2399h
```

Make sure that `~/traefik/traefik-config.toml` contains the following:
```
[[tls.certificates]]
  certFile = "/certs/wikijs.crt"
  keyFile = "/certs/wikijs.key"
```

Copy laboratoriator's `containers/wikijs.docker-compose.yml` to your host's home directory and run it.
```
$ docker compose -f wikijs.docker-compose.yml up -d
```

### Install Nextcloud

Create a certificate/key for Nextcloud and save it inside `certs`.

```
$ step ca certificate --ca-url https://ca.moxthos.art:8443 nextcloud.moxthos.art certs/nextcloud.crt certs/nextcloud.key --not-after 2399h
```

Make sure that `~/traefik/traefik-config.toml` contains the following:
```
[[tls.certificates]]
  certFile = "/certs/nextcloud.crt"
  keyFile = "/certs/nextcloud.key"
```

Copy laboratoriator's `containers/nextcloud.docker-compose.yml` to your host's home directory and run it.

```
$ docker compose -f nextcloud.docker-compose.yml up -d
```

### Install Rocketchat

Create a certificate/key for Rocketchat and save it inside `certs`.

```
$ step ca certificate --ca-url https://ca.moxthos.art:8443 rocketchat.moxthos.art certs/rocketchat.crt certs/rocketchat.key --not-after 2399h
```

Make sure that `~/traefik/traefik-config.toml` contains the following:
```
[[tls.certificates]]
  certFile = "/certs/rocketchat.crt"
  keyFile = "/certs/rocketchat.key"
```

Copy laboratoriator's `containers/rocketchat.docker-compose.yml` to your host's home directory and run it.

```
$ docker compose -f rocketchat.docker-compose.yml up -d
```

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- ROADMAP -->
## Roadmap

- [x] Step installation
- [x] Traefik installation
- [x] Gitea installation
- [x] WikiJS installation
- [x] Nextcloud installation
- [x] Rocketchat installation

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

If you have a suggestion that would make this better, please fork the repo and create a pull request. You can also simply open an issue with the tag "enhancement".
Don't forget to give the project a star! Thanks again!

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE` for more information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTACT -->
## Contact

Christos Kaltsas - me@christoskaltsas.com

Project Link: [https://github.com/bQqqr/laboratoritator](https://github.com/bQqqr/laboratoritator)

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/bQqqr/laboratoritator.svg?style=for-the-badge
[contributors-url]: https://github.com/bQqqr/laboratoritator/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/bQqqr/laboratoritator.svg?style=for-the-badge
[forks-url]: https://github.com/bQqqr/laboratoritator/network/members
[stars-shield]: https://img.shields.io/github/stars/bQqqr/laboratoritator.svg?style=for-the-badge
[stars-url]: https://github.com/bQqqr/laboratoritator/stargazers
[issues-shield]: https://img.shields.io/github/issues/bQqqr/laboratoritator.svg?style=for-the-badge
[issues-url]: https://github.com/bQqqr/laboratoritator/issues
[license-shield]: https://img.shields.io/github/license/bQqqr/laboratoritator.svg?style=for-the-badge
[license-url]: https://github.com/bQqqr/laboratoritator/blob/master/LICENSE
[docker-installation-url]: https://docs.docker.com/engine/install/debian/#install-using-the-repository 
[docker-compose-installation-url]: https://docs.docker.com/compose/install/linux/#install-using-the-repository
[lazydocker-installation-url]: https://github.com/jesseduffield/lazydocker#installation
[step-installation-url]: https://smallstep.com/docs/step-ca/installation#linux-packages-amd64
