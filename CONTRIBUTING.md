# Contributing to Blueprint Skills

Thanks for considering a contribution!

## How to Contribute

### Reporting Issues
- Use GitHub Issues
- Include: skill name, expected behavior, actual behavior, reproduction steps

### Proposing New Skills
1. Open an issue describing the skill, its purpose, and trigger conditions
2. Wait for discussion before submitting a PR
3. Each new skill must:
   - Solve a real, repeatable problem
   - Not overlap with existing skills (or improve them clearly)
   - Follow the SKILL.md template
   - Include at least one usage example

### Pull Requests
1. Fork the repo
2. Create a branch: `git checkout -b skill/your-skill-name`
3. Add your skill in `skills/your-skill-name/SKILL.md`
4. Update README.md if adding a new skill
5. Submit PR with description of changes

## Skill Quality Standards

- Clear trigger conditions
- Imperative language ("Always do X", not "You might want to consider X")
- Concrete examples
- Anti-patterns documented
- No more than 200 lines per SKILL.md (longer = should be split)

## Code of Conduct

Be respectful. Assume good intent. Focus on the code, not the person.

## License

By contributing, you agree your contributions are licensed under MIT.
