# DHCP and DNS lab with dnsmasq: a networking lab for a classroom, in one VM

[![check](https://github.com/mitsurutech/lab-dhcp-dns/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-dhcp-dns/actions/workflows/check.yml)

A ready-made **DHCP and DNS lab** for teaching how a network hands out addresses and answers names, from the [MitsuruTech Learning Platform](https://mitsurutech.co.uk/learning), where schools and colleges give every student their own Ubuntu VM with a terminal in the browser. Each student gets a small network built inside their machine: a server and two clients, `pc1` and `pc2`, that start with **no address at all**. The student configures **dnsmasq**, the small DHCP and DNS server found in home routers and labs everywhere, so that the server leases addresses from a range, tells the clients their router and DNS server, gives one client a **fixed address by its MAC address**, and answers for the lab's own names. Nothing is installed on the student's computer.

The server and the clients are **network namespaces** on one virtual switch, and the real tools do the work: `dnsmasq` on the server, `dhclient` on the clients, `dig`, `ip` and `tcpdump` to look at it. That is the point.

## The lab

| Lab | File | What the student does | Help page |
| --- | --- | --- | --- |
| DHCP and DNS | [`dhcp-dns.lab.yaml`](dhcp-dns.lab.yaml) | Write the dnsmasq configuration so the clients are leased addresses from a range with the right router and DNS server, one client always gets the same address, and the lab's two names resolve and nothing else does | [DHCP and DNS lab](https://mitsurutech.co.uk/help/lab-dhcp-dns) |

The lab leaves a README in the student's home folder that says what to do and gives none of the answers. It has a **check**, a script that marks a handed-in VM without the teacher reading it. It runs on a copy of the saved machine after a fresh start, stops whatever dnsmasq was running, starts one from the saved files in `/etc/dnsmasq.d`, wipes both clients and has them ask again. So it passes only if the real behaviour holds: the clients are leased addresses in the range with the right router, DNS server and lease time, the second client gets its fixed address, the two names resolve to the right addresses, and an unknown name is not answered. Addresses typed by hand on the clients, a wildcard answer for every name, or names put in `/etc/hosts` do not pass.

## What is in the file

`dhcp-dns.lab.yaml` is a whole **VM lab definition**, exactly what the platform runs:

| Key | What it is |
| --- | --- |
| `cpus`, `memory`, `disk` | the size of each student's VM: 1 CPU, 1024 MB and 10 GB |
| `open` | the ports that open in the browser: none, the lab is the terminal |
| `setup` | a script run once, as root, the first time each student's VM starts: it installs dnsmasq, the DHCP client, `dig` and `tcpdump` from Ubuntu's archive, writes the `dnslab` command, the lab's README and the unit that rebuilds the network at boot |
| `check` | the marking script, run as root on a copy of the saved disk after a fresh boot; its exit status is the result and `0` passes |

The `dnslab` command it installs: `dnslab status` lists the machines, `dnslab restart` runs dnsmasq again from the saved configuration, `dnslab renew pc1` makes a client ask for a lease and `dnslab shell pc1` opens a shell inside a machine.

## Using it on the platform

**To set it as work for a class:**

1. In your module, add a task and choose the **Ubuntu VM lab** card.
2. Press the sample, or paste the contents of the file here. Using a sample replaces the text, and Undo brings yours back.
3. Publish and try your own copy first. Every student gets a VM of their own, kept between sessions.
4. Assign it like any task. When it is handed in, run the check on a student's row in the class list: it starts a copy of their saved machine that is not saved back, and shows whether it passed and what it printed.

The help manual's [guide to Ubuntu VM labs](https://mitsurutech.co.uk/help/labs) and its [sample labs page](https://mitsurutech.co.uk/help/sample-labs) cover the rest, and a basic lesson for this lab, **DHCP and DNS Basics**, is in the Public college and in every new college. To change the lab, copy the file and edit the addresses and names in the `setup` and the `check` together.

No college account yet? [Your first year is free](https://mitsurutech.co.uk/pricing) for up to 50 students, and [signing up](https://mitsurutech.co.uk/signup) takes an email address.

## Checking the file

On every push, `.github/workflows/check.yml` checks that the file is valid YAML with the five keys above and that `setup` and `check` are each inside the platform's 16 KB limit. It does not boot a VM: that is done on the platform's own machines.

## What does not work in a lab

The VM reaches Ubuntu's package archive, PyPI, npm and GitHub, and nothing else, so a lab cannot be pointed at a host on the internet, and the lab's DNS server has nowhere to forward a name to. It is one VM: the server and the clients are namespaces inside it, so the clients are not separate machines, and there is no second DHCP server to compete with.

## About

Made by [Mitsuru Technologies](https://mitsurutech.co.uk) for the [MitsuruTech Learning Platform](https://mitsurutech.co.uk/learning): Python, Java, Go, Node, SQL and Linux in the browser for schools and colleges, with container labs, VM labs, web apps deployed from GitHub, and quizzes that mark themselves. Try code with no account in the [playground](https://mitsurutech.co.uk/playground).

More labs from the same place: [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea) and [lab-sample](https://github.com/mitsurutech/lab-sample).

## Licence

The lab definition, the `dnslab` tool and this documentation are MIT licensed, see [LICENSE](LICENSE): copy them, change them, teach with them. [dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) is GPL-2.0 or GPL-3.0 licensed and the ISC DHCP client is MPL-2.0 licensed; both are installed from Ubuntu's archive when a VM starts, and nothing of them is distributed here.
