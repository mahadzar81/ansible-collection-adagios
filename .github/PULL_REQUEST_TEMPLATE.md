---
## Description
<!-- What does this PR change and why? -->

## Type of change
- [ ] Bug fix
- [ ] New feature
- [ ] SELinux policy update
- [ ] Idempotency fix
- [ ] Documentation update
- [ ] CI/CD update

## Checklist
- [ ] `yamllint .` passes locally
- [ ] `ansible-lint` passes locally
- [ ] `molecule test` passes locally (or justification for skip)
- [ ] Second `molecule converge` run shows **0 changed** tasks (idempotency)
- [ ] `galaxy.yml` version bumped if this is a releasable change
- [ ] `CHANGELOG.md` updated
- [ ] README updated if new variables or behaviour introduced

## Testing notes
<!-- How was this tested? Which OS/SELinux mode? -->
