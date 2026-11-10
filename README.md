# Computing Systems Security

Coursework for **Computing Systems Security (SSC)**, MSc in Informatics Engineering, University of Minho, academic year 2026/2027.

The course looks at a machine the way both an attacker and a defender do. The blocks use the same isolated lab virtual machine throughout: the work starts with enumerating its attack surface and modelling its threats, and later blocks study the mechanisms behind what was found and how to harden the machine.

## Lab environment

For Blocks 1 to 4, the course provided a lab virtual machine that every student runs locally:

| | |
|---|---|
| Image | SSC lab base image v1.0 |
| Operating system | Ubuntu 24.04 LTS, amd64 |
| Hostname | `ssc-lab` |
| Hypervisor | VirtualBox, isolated from other networks |

The image is deliberately vulnerable: it ships with planted weaknesses and fake credentials for the exercises. All the labs run on this same machine, and some start extra services inside it:

- **Lab 4:** a local OpenID Provider ([dex](https://dexidp.io/)) at `localhost:5556` and a demo client at `localhost:3000`.
- **Lab 5:** two Docker containers, `jump` and `target`.

## Repository structure

```
computing-systems-security-uminho/
├── README.md                                  
├── .gitignore
├── block-1/
│   ├── README.md
│   ├── Report-Block1.pdf
│   ├── lab-01-environment-and-attack-surface/   
│   ├── lab-02-threat-modelling/
│   └── lectures/                               
├── block-2/
│   ├── README.md
│   ├── Report-Block2.pdf
│   ├── lab-03-pam-and-the-authentication-stack/
│   ├── lab-04-oidc-and-federated-identity/
│   ├── lab-05-ssh-certificates-and-hardening/
│   └── lectures/
├── block-3/
│   └── README.md
├── block-4/
│   └── README.md
└── capstone/
    └── README.md
```

Each block folder has a `README.md` with a summary of the work and a link to the report, which is the assessed deliverable. Each lab folder has its own `README.md` with what I did, the key commands and the findings, next to the lab handout.

## Contents

| Block | Topic | Labs | Status |
|---|---|---|---|
| [Block 1](block-1/) | Environment setup, attack surface enumeration and threat modelling | 1–2 | ✅ Done |
| [Block 2](block-2/) | Authentication and authorisation: PAM, OIDC and SSH certificates | 3–5 | ✅ Done |
| Block 3 | To be added | | ⏳ Coming soon |
| Block 4 | To be added | | ⏳ Coming soon |
| Capstone | To be added | | ⏳ Coming soon |

## Notes

- **Isolated lab environment.** All work is done inside the isolated course lab VM and its containers. No system outside that environment is targeted.
- **Fake credentials.** Any credentials shown in screenshots are fake values planted by the course lab image, or keys and tokens generated inside the lab for the exercises. They are not real secrets.
- **Course material.** Each lab folder includes the lab handout, and each block's `lectures/` folder the lecture slides. They were written by the course lecturer, who holds the rights to them, and are included here for reference.
- **Shared as a portfolio.** This repository documents my own coursework. If you are taking this course, follow its academic-integrity rules.

## Author

Marco António Ferreira Abreu — MSc in Informatics Engineering, University of Minho
