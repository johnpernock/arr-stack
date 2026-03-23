# Contributing to arr-stack Documentation

Thank you for considering contributing to this project! This guide is meant to help everyone set up and maintain their *arr stack on Unraid, and community contributions make it better.

## How to Contribute

### Reporting Issues

Found something wrong or confusing?

1. Check [existing issues](https://github.com/johnpernock/arr-stack/issues) first
2. If it's new, open an issue with:
   - Clear description of the problem
   - What documentation section it relates to
   - Steps to reproduce (if applicable)
   - Your setup details (Unraid version, app versions, etc.)

### Suggesting Improvements

Have an idea to make the guides better?

1. Open an issue with the "enhancement" label
2. Describe what you'd like to see
3. Explain why it would be helpful
4. If possible, suggest where it should go in the docs

### Contributing Documentation

Want to add or improve documentation?

#### Quick Fixes

For typos, broken links, or small clarifications:

1. Click "Edit" on the file in GitHub
2. Make your change
3. Submit a pull request
4. That's it!

#### Larger Contributions

For new sections, major rewrites, or new guides:

1. **Fork the repository**
2. **Create a feature branch**
   ```bash
   git checkout -b feature/my-improvement
   ```
3. **Make your changes**
   - Follow the writing style (see below)
   - Test any code or commands
   - Add to table of contents if needed
4. **Commit with clear messages**
   ```bash
   git commit -m "Add section on custom Jackett indexers"
   ```
5. **Push to your fork**
   ```bash
   git push origin feature/my-improvement
   ```
6. **Open a Pull Request**
   - Describe what you changed and why
   - Link to any related issues
   - Be open to feedback

## Writing Style Guide

To keep documentation consistent:

### Tone and Voice

- **Conversational but technical** - Explain like you're helping a friend
- **Assume intelligent reader** - Don't over-explain basics, but don't assume too much knowledge
- **Be encouraging** - This stuff can be confusing, be supportive
- **Honest about complexity** - Don't sugarcoat, but show it's achievable

### Structure

- **Clear headings** - Use hierarchy (H2, H3, H4)
- **Short paragraphs** - 2-4 sentences max
- **Code blocks** for commands, configs, regex
- **Tables** for comparisons or options
- **Examples** whenever possible
- **Warnings** in bold for critical info

### Formatting Conventions

**Commands:**
```bash
# Use code blocks with bash highlighting
chmod -R 777 /mnt/user/data/
```

**File paths:**
Use inline code: `/mnt/user/data/media/`

**Application names:**
First mention: **Sonarr** (bold)
After: Sonarr (normal)

**Settings navigation:**
Use arrows: Settings → Download Clients → Add

**Emphasis:**
- **Bold** for important concepts, warnings
- *Italic* sparingly, only for subtle emphasis
- `Code formatting` for: commands, paths, values, technical terms

**Lists:**
- Bullet points for unordered
- Numbers for sequential steps
- Checkboxes for todo items

### Examples to Include

Good documentation includes:

- **Real-world examples** (like the Ultra.cc walkthrough)
- **Before/after** scenarios
- **Common mistakes** and how to avoid them
- **Expected output** for commands
- **Troubleshooting** for each major section

### What to Avoid

- **Don't use "simply" or "just"** - What's simple for you may not be for others
- **Avoid assumptions** - State requirements explicitly
- **No broken links** - Check all links work
- **No personal opinions** unless clearly labeled as such
- **Avoid outdated info** - Mark time-sensitive content clearly

## Types of Contributions Needed

### High Priority

- **Updates for app changes** - When Sonarr/Radarr/etc. change
- **Troubleshooting additions** - New issues you've encountered and solved
- **Clarifications** - Parts that confused you
- **Missing steps** - Gaps in the guides

### Nice to Have

- **Alternative approaches** - Different ways to achieve the same goal
- **Advanced configurations** - Power user setups
- **Integration guides** - Connecting with other tools
- **Screenshots** - Visual aids (must be clean and up-to-date)

### Always Welcome

- **Typo fixes**
- **Broken link fixes**
- **Grammar improvements**
- **Better examples**
- **Clearer explanations**

## Testing Your Changes

Before submitting:

1. **Proofread** - Check spelling and grammar
2. **Test commands** - If you added bash commands, verify they work
3. **Test links** - Click all links to verify they work
4. **Check formatting** - Preview in Markdown to ensure it renders correctly
5. **Verify accuracy** - Especially for technical details

## What Happens After You Contribute

1. **Review** - I'll review your pull request
2. **Discussion** - Might ask questions or suggest changes
3. **Refinement** - Make any requested changes
4. **Merge** - Once approved, it'll be merged
5. **Credit** - You'll be credited in the commit and release notes

## Documentation Structure

Understanding the structure helps you know where to add content:

```
arr-stack/
├── README.md                          # Overview and quick start
├── QUICK_START.md                    # Cheat sheet for fast reference
├── LICENSE                           # MIT License
├── CONTRIBUTING.md                   # This file
└── docs/
    ├── setup-guide.md                # Complete installation walkthrough
    ├── quality-profiles-guide.md     # Quality control and filtering
    ├── troubleshooting.md            # Common issues and solutions
    ├── seedbox-examples.md           # Seedbox-specific configurations
    └── maintenance.md                # Ongoing maintenance tasks
```

### When to Add to Each File

**README.md** - Overview only, link to detailed docs

**QUICK_START.md** - Reference material, not tutorials

**setup-guide.md** - Step-by-step installation and initial config

**quality-profiles-guide.md** - Anything about filtering, scoring, custom formats

**troubleshooting.md** - Problem/solution pairs

**seedbox-examples.md** - Seedbox-specific content

**maintenance.md** - Ongoing care and updates

**New file** - If topic is substantial and doesn't fit existing structure

## Code of Conduct

### Be Respectful

- Respect different skill levels
- Be patient with questions
- Constructive criticism only
- No harassment or discrimination
- Assume good intent

### Be Helpful

- Answer questions kindly
- Share knowledge freely
- Help newcomers
- Acknowledge contributions

### Be Collaborative

- Listen to feedback
- Discuss disagreements professionally
- Focus on the best outcome for users
- Give credit where due

## Recognition

Contributors are recognized in:

- Commit history (preserved forever)
- Release notes (for significant contributions)
- Project README (for major contributions)

## Questions?

Not sure about something?

- **Open an issue** to ask
- **Start a discussion** in GitHub Discussions
- **Check existing issues/PRs** for similar questions

## Thank You!

Every contribution, no matter how small, helps others in the community. Thank you for making this documentation better!

---

**Ready to contribute?** Fork the repo and start improving!
