# Minimal image for the CI smoke test.
#
# FROM scratch on purpose: it contains no OS packages, so the Trivy step in the
# action has a deterministic, permanently-clean target. A distro base image
# would let an unrelated CVE published upstream turn this job red and tell us
# nothing about the action itself.
FROM scratch
COPY hello.txt /hello.txt
