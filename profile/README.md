# Welcome!

Our intention with this project is to increase access to CHERI-based systems and platforms, respectively CheriBSD and the ARMv8-based Morello research prototype, by making ephemeral self-hosted GitHub runners available for them.

Should you have any questions, please make use of the [Discussions][discussions] section.

Thank you!

## Quick start

This GitHub organisation provides its members with access to jailed runners, on either a cluster of racked, quad-core Morello boards, each with 64GB RAM, or single-threaded QEMU instances better suited to unit testing and fuzzing. It also provides forks of Digital Catapult's documentation, tooling, and [Packer][packer] builds, so that you can recreate similar runners within your own infrastructure should your projects require access to proprietary or confidential information.

If granted access, you will need to define the workflow for a private repository. There are constraints on member privileges, though few on the structure of your project; it could be a novel, stand-alone C/C++ codebase or a pull from an existing repository.

## C/C++ spatial memory safety

```c
#include <stdio.h>
#include <string.h>

int
main(void)
{
    char buf[9];
    strcpy(buf, "123456789");
    printf("%s\n", buf);
}
```

This example triggers the following during compilation: `warning: 'strcpy' will always overflow; destination buffer has size 9, but the source string has length 10 (including NUL byte)`, which is promising for our demonstration. One aspect to CHERI's spatial memory safety is the raising of exceptions when such overflows do occur: `In-address space security exception (core dumped)`, a kind of failure that becomes readily identifiable via GitHub Actions' UI.

In the accompanying GitHub workflow, you must specify the correct system to run on. Either string is sufficient: `cheribsd` or `cheribsd-23.11`. Unlike FreeBSD, all CheriBSD releases use their date to signify the version.

```yaml
name: Buffer overflow
on: push

jobs:
  workflow:
    runs-on: cheribsd-23.11
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Install toolchain
        run: pkg64 install -y llvm-base

      - name: Compile exercise
        run: |
          cc -Wall -o buffer-overflow buffer-overflow.c -mabi=purecap -march=morello

      - name: Analyse file
        run: file ./buffer-overflow

      - name: Run binary
        run: ./buffer-overflow
```

## Further reading

To benefit from CHERI further, it would also be worthwhile reviewing the assumptions, functions, and examples in the following: the [_CHERI C/C++ Programming Guide_][guide] and [_Adversarial CHERI Exercises and Missions_][adversarial].

## Acknowledgements

Digital Catapult owes a special thanks to the following organisations: UKRI for enabling this project as part of the Digital Security by Design challenge, and the University of Cambridge and SRI International for input, patches, and the provisioning of physical infrastructure.

<!-- Links -->
[packer]: https://github.com/dc-dsbd/dsbd-github-baseimage
[guide]: https://www.cl.cam.ac.uk/techreports/UCAM-CL-TR-947.pdf
[adversarial]: https://ctsrd-cheri.github.io/cheri-exercises/
[discussions]: https://github.com/orgs/dc-dsbd/discussions
