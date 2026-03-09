---
name: "super-optimizer"
description: "AI-powered writing optimizer for technical documentation, code comments, and prose"
version: "1.2.1"
author: "OpenClaw Team"
tags:
  - writing
  - ai
  - automation
  - optimization
  - technical-writing
maintainer: "sysadmin@openclaw.io"
type: "skill"
category: "content"
license: "MIT"
dependencies:
  - "openai>=1.0.0"
  - "anthropic>=0.8.0"
  - "tiktoken>=0.5.0"
  - "python-dotenv>=1.0.0"
  - "rich>=13.0.0"
required_apis:
  - "openai"
  - "anthropic"
os:
  - "linux"
  - "macos"
  - "windows"
---

# Super Optimizer

## Purpose

Super Optimizer is an AI-powered writing assistant that transforms raw, verbose, or poorly structured text into clear, concise, and professional content. It's designed for:

**Real Use Cases:**

1. **Technical Documentation Cleanup**: Convert developer notes into polished documentation. Example: Take a 500-word informal explanation of a function and output a structured doc with parameters, returns, and examples.

2. **Code Comment Optimization**: Rewrite inline comments to be meaningful and consistent. Example: Transform `"// fix this later"` into `"// TODO: Handle edge case where user_id is null - see issue #234"` or remove non-actionable comments entirely.

3. **Commit Message Enhancement**: Take informal git commit messages and rewrite them to follow Conventional Commits format. Example: `"fixed bug"` → `"fix(auth): prevent null pointer when token expires"` 

4. **Email Draft Refinement**: Convert rough email drafts into professional, actionable messages with proper tone and structure, preserving key information while removing filler words.

5. **API Documentation Generation**: Extract implicit information from code and generate complete OpenAPI/Swagger documentation with accurate descriptions.

6. **README Optimization**: Restructure and clarify README files to follow industry best practices (badges, installation, usage examples, contribution guidelines).

7. **Blog Post Polish**: Improve grammar, flow, and SEO while maintaining the author's voice and technical accuracy.

8. **Requirement Specification**: Convert meeting notes or brainstorming sessions into structured PRD/Technical Requirement documents with clear acceptance criteria.

## Scope

### Supported Commands

**Primary:** `openclaw optimize [OPTIONS] <FILE|STDIN>`

**Aliases:** `openclaw opt`, `openclaw improve`, `openclaw refine`

**Command Options:**

```bash
# Optimize a file in-place
openclaw optimize --type=docs src/docs/api.md

# Pipe from stdin, output to stdout (dry-run)
cat draft.txt | openclaw optimize --type=prose --dry-run

# Optimize with specific style guide
openclaw optimize --type=docs --style=google src/README.md

# Process multiple files with pattern
openclaw optimize --type=code "src/**/*.py"

# Use specific provider/model
openclaw optimize --provider=anthropic --model=claude-3-5-sonnet-20241022 draft.md

# Apply conservative changes only (minimal rewriting)
openclaw optimize --conservative commit_msg.txt

# Verbose output with token usage
openclaw optimize --verbose --show-cost technical_spec.md

# Parallel processing of multiple files
openclaw optimize --parallel=4 --type=docs docs/*.md

# Exclude certain optimization rules
openclaw optimize --no-fix-grammar --no-rephrase proposal.txt
```

**Secondary Commands:**

```bash
# Preview changes without applying
openclaw optimize --diff document.md

# Batch process with configuration file
openclaw optimize --config .optimizerc.json docs/

# List available optimization profiles
openclaw optimize --list-profiles

# Validate optimization configuration
openclaw optimize --validate-config .optimizerc.json

# Estimate cost before processing
openclaw optimize --estimate-cost large_document.txt

# Rollback last optimization (git-based)
openclaw optimize --rollback document.md
```

## Detailed Work Process

### Phase 1: File Analysis

1. **Input Processing**
   - Read file from path or stdin
   - Detect encoding (UTF-8, ISO-8859-1)
   - Parse file type from extension or shebang
   - For binaries: reject with error

2. **Content Profiling**
   - Calculate statistics: word count, sentence length variance, Flesch Reading Ease score
   - Detect technical terms, jargon density
   - Identify structure (headings, lists, code blocks, tables)
   - Flag potential issues: passive voice, weak verbs, filler words (very, really, basically)

3. **Context Gathering** (if available)
   - Check for adjacent files (e.g., if optimizing `api.py`, look for `api_test.py` or `api.md`)
   - Load project-specific glossary from `.optimizer/glossary.yaml`
   - Detect programming language and apply language-specific rules
   - Check for existing lint/test configurations to align with project standards

### Phase 2: Strategy Selection

1. **Type Detection**: Default from file extension or explicit `--type` flag:
   - `docs`: Markdown, reStructuredText, documentation
   - `code`: Source files with comments
   - `prose`: General articles, emails, blogs
   - `commit`: Git commit messages
   - `api`: OpenAPI/Swagger specs

2. **Style Selection**:
   - `default`: Balanced, professional tone
   - `google`: Google Developer Documentation Style
   - `microsoft`: Microsoft Writing Style Guide
   - `concise`: Maximum brevity (technical tweets, changelogs)
   - `academic`: Formal, citation-aware
   - `custom`: Loaded from `.optimizerc.json`

3. **Aggression Level**:
   - `conservative` (--conservative): Only fix grammar/spelling, minimal rewrites
   - `moderate`: Restructure sentences, improve flow, standard moderate changes
   - `aggressive`: Full rewrite where needed, restructure paragraphs (default)

### Phase 3: Optimization Execution

1. **Chunking Strategy** (for large files >100KB):
   - Split on natural boundaries (headers, code blocks, paragraphs)
   - Maintain overlap of 1-2 sentences between chunks for context
   - Process chunks in parallel if `--parallel=N` set
   - Assemble results preserving original structure

2. **AI Processing**:
   - Construct prompt with:
     * Original text
     * Detected type and style rules
     * Specific instructions based on type (e.g., "Convert to Conventional Commits format")
     * Project glossary terms to preserve
     * Examples of good/bad transformations
   - Call selected provider/model with stream=False, temperature=0.2 for consistency
   - Implement fallback chain: primary → secondary provider on rate limit/error
   - Track token usage for cost reporting

3. **Post-Processing**:
   - Parse AI response, extract only the optimized text (strip conversational preamble)
   - Validate: check that code blocks are unchanged, technical terms preserved
   - Ensure file structure (headings, code fences, formatting) maintained
   - Apply diff algorithm to generate unified diff

### Phase 4: Output & Integration

1. **Output Mode**:
   - Default: In-place replacement with git staging
   - `--diff`: Display colored diff to stdout, no changes
   - `--stdout`: Write result to stdout
   - `--backup`: Create `filename.optimized.bak` before writing

2. **Git Integration**:
   - Check if file tracked by git
   - If yes: stage changes automatically
   - If dirty working tree: warn and require `--force`
   - Create annotated tag `optimized/<timestamp>/<filename>` for rollback

3. **Reporting**:
   - Summary: "Optimized X words, saved Y%, improved readability score from Z to W"
   - Token count and cost estimate
   - Warnings: "Review changed technical terms: X, Y, Z"
   - Exit codes: 0=success, 1=validation error, 2=API error, 3=file error

## Golden Rules

1. **Never modify code logic**: Only modify comments, strings, and documentation. If code changes detected, abort with error.

2. **Preserve exact technical terms**: Use project glossary to whitelist terms that must not be altered (function names, class names, API endpoints, config keys).

3. **Code block integrity**: Triple-check that everything inside ```lang``` blocks or indented code is byte-for-byte identical. If changes detected, reject the entire optimization.

4. **No hallucination**: Do not add information not present in original. If context needed, mark with `[NEEDS CONTEXT]` comment, don't invent.

5. **Maintain voices**: In collaborative docs, preserve attribution (e.g., "As Alice mentioned...") and don't standardize voices.

6. **Security first**: Never optimize files in:
   - `.env`, `*.key`, `*.pem`, `credentials.json`, `secrets/`
   - Auto-detect and abort with security warning

7. **Backup requirement**: Always create backup unless `--no-backup` explicitly set. Use git tags if available, otherwise `.bak` files.

8. **Respect existing lint**: If project uses specific style (checked via `.eslintrc`, `.stylelintrc`, `pyproject.toml`), adapt to those conventions.

9. **Exit on uncertainty**: If confidence in optimization < 85% (measured via AI response), output warning and skip file.

10. **Cost transparency**: Always report token usage and estimated cost. Use `--estimate-cost` for large batches before execution.

## Examples

### Example 1: Optimize Markdown Documentation

**Input** (`docs/getting-started.md`):
```markdown
# Getting Started

So yeah basically to get started you gotta install the thing. Just do npm install and then you can run it. Its pretty easy.

## Installation

Just run this command:

```
npm install @flickclaw/sdk
```

After that you can import it.

## Usage

Heres a simple example:
```

**Command:**
```bash
openclaw optimize --type=docs --style=google docs/getting-started.md
```

**Output** (with diff preview):
```diff
 --- a/docs/getting-started.md
 +++ b/docs/getting-started.md
 @@ -1,12 +1,30 @@
 -# Getting Started
-
-So yeah basically to get started you gotta install the thing. Just do npm install
-and then you can run it. Its pretty easy.
+# Getting Started
+
+This guide helps you install and configure FlickClaw SDK in minutes.
+
+## Prerequisites
+
+- Node.js 18+
+- npm 9+
+- FlickClaw account with API key
+
+## Installation
+
+Install the SDK using npm:
+
+```bash
+npm install @flickclaw/sdk
+```
+
+## Configuration
+
+Create a `.env` file in your project root:
+
+```bash
+FLICKCLAW_API_KEY=your_api_key_here
+FLICKCLAW_ENV=production
+```
+
+## Quick Start
+
+Import and initialize the client:
+
+```javascript
+import { FlickClaw } from '@flickclaw/sdk';
+
+const client = new FlickClaw({
+  apiKey: process.env.FLICKCLAW_API_KEY
+});
+```
```

**Console Output:**
```
✓ Optimized docs/getting-started.md
  • Words: 45 → 89 (98% increase in information density)
  • Readability: College → 9th grade
  • Cost: $0.0023 (1,234 tokens)
  • Changes committed to git
```

### Example 2: Optimize Git Commit Message

**Input** (`git commit -m "msg.txt"`):
```
fixed auth bug and updated deps
```

**Command:**
```bash
openclaw optimize --type=commit --stdout <<< "fixed auth bug and updated deps"
```

**Output:**
```
fix(auth): prevent null pointer when token expires

- Add null check for expired tokens in AuthMiddleware
- Handle TokenExpiredError gracefully with user-friendly message
- Update axios to 1.6.0 for security patches

Closes #234

BREAKING CHANGE: None
```

### Example 3: Code Comment Cleanup

**Input** (`src/auth.js`):
```javascript
function validateToken(token) {
  // check if token is good
  if (!token) return false;
  // todo: fix this
  const decoded = jwt.decode(token);
  // this is the expiration check thing
  if (decoded.exp < Date.now()) return false;
  // ok good
  return true;
}
```

**Command:**
```bash
openclaw optimize --type=code --conservative src/auth.js
```

**Output** (in-place):
```javascript
function validateToken(token) {
  // Validate token exists
  if (!token) return false;

  // TODO: Implement refresh token flow when token expires
  const decoded = jwt.decode(token);

  // Check expiration (exp is in seconds, Date.now() in milliseconds)
  if (decoded.exp * 1000 < Date.now()) return false;

  return true;
}
```

**Changed markers:**
```
✓ src/auth.js optimized
  • Fixed grammar in 3 comments
  • Added clarification to expiration check
  • Preserved all code logic (23 lines unchanged)
```

### Example 4: Batch Process with Profile

**Config** (`.optimizer/profiles/technical.json`):
```json
{
  "name": "technical",
  "type": "docs",
  "style": "concise",
  "rules": {
    "max_sentence_length": 25,
    "avoid_passive": true,
    "required_sections": ["Parameters", "Returns", "Examples"],
    "fix_grammar": false,
    "rephrase": true
  },
  "glossary": ["FlickClaw", "SDK", "API", "RESTful", "OAuth2"]
}
```

**Command:**
```bash
openclaw optimize --profile=technical docs/api/*.md
```

**Result:** Processes all markdown files in `docs/api/` using the technical profile, enforcing concise style, adding required sections if missing, and preserving glossary terms.

### Example 5: Dry-Run with Cost Estimation

**Command:**
```bash
openclaw optimize --type=prose --estimate-cost --parallel=4 --verbose draft_blog.txt
```

**Output:**
```
📊 Cost Estimation for: draft_blog.txt
   Size: 45 KB (12,345 words)
   Estimated chunks: 13 (each ~3KB)
   Model: claude-3-5-sonnet-20241022
   Input tokens: ~45,000
   Output tokens: ~40,000 (estimated)
   Total cost: $0.0685
   ⏱️  Estimated time: 2m 34s with 4 workers

Use --execute to proceed with optimization.
```

## Verification

### Post-Optimization Validation

1. **Spell Check**: Run `codespell` or `cspell` on optimized files:
   ```bash
   codespell optimized_file.md
   # Should return no errors
   ```

2. **Diff Review**: Check that only intended sections changed:
   ```bash
   git diff --word-diff=color optimized_file.md | grep -E '\{-.*-\}|{\+.*\+}'
   # Review changes, ensure no code/keyword mangling
   ```

3. **Readability Score**: Compare Flesch Reading Ease before/after:
   ```bash
   # Before (original)
   style-ease original.md  # Target: 60+ for docs
   # After (optimized)
   style-ease optimized.md  # Should improve by 10+ points
   ```

4. **Markdown Validation**: Ensure optimized markdown is valid:
   ```bash
   markdown-cli validate optimized.md
   # or
   mdlint optimized.md
   ```

5. **Test Suite**: If optimizing code comments, ensure tests still pass:
   ```bash
   npm test  # or pytest, etc.
   # Should pass, since code logic unchanged
   ```

6. **Link Check**: For optimized documentation, verify internal links:
   ```bash
   markdown-link-check optimized.md
   # All links should resolve
   ```

### Automated Verification Script

Create `.optimizer/verify.sh`:
```bash
#!/bin/bash
set -e

FILE="$1"

# 1. No code changes
if git diff --quiet "$FILE"; then
    echo "✓ No unstaged changes detected"
else
    echo "✗ Unstaged changes found in $FILE"
    git diff --name-only | head -5
    exit 1
fi

# 2. Grammar check
if command -vvale &>/dev/null; then
    vale "$FILE" || echo "⚠ Vale warnings - review manually"
fi

# 3. Length constraints
WORDS=$(wc -w < "$FILE")
if [ "$WORDS" -lt 10 ]; then
    echo "✗ File too short ($WORDS words) - possible over-optimization"
    exit 1
fi

echo "✓ Verification passed for $FILE"
```

Run automatically after optimization with `--post-hook .optimizer/verify.sh`.

## Rollback

### Rollback Methods

**1. Git Rollback** (if optimized files were committed):
```bash
# List optimization tags
git tag | grep optimized/

# Rollback specific file to pre-optimization state
git checkout optimized/20260309_142345/src/auth.js -- src/auth.js

# Or revert the entire optimization commit
git revert <commit_hash>
# Then resolve any merge conflicts manually
```

**2. Backup File Restore** (if `--backup` used):
```bash
# Restore from .bak file
cp src/auth.js.optimized.bak src/auth.js

# Or if timestamped backup
ls -la src/*.bak
cp src/auth.js.bak_20260309_1423 src/auth.js
```

**3. Manual Rollback via Diff** (if no backup):
```bash
# If you still have the diff output saved
patch -R < optimization_changes.diff

# Or use git reflog to find previous version
git reflog | grep "optimize"
git checkout HEAD@{2} -- src/auth.js  # specific reflog entry
```

**4. Super Optimizer Built-in Rollback**:
```bash
# Rollback last optimization for specific file
openclaw optimize --rollback src/auth.js

# Rollback all optimized files in directory
openclaw optimize --rollback-all docs/

# List available rollback points
openclaw optimize --list-rollbacks
```

### Rollback Safety Checks

Before any rollback operation:
1. **Confirm changes**: Show diff between current and backup state
2. **Check dependencies**: Ensure rollback won't break imports/exports
3. **Preserve work**: If file modified after optimization, merge changes rather than overwrite
4. **Log action**: Record rollback in `.optimizer/rollback.log` with:
   ```
   2026-03-09 14:45:12 ROLLBACK src/auth.js
   Reason: Over-optimization removed important nuance
   Restored from: tag optimized/20260309_142345
   ```

### Automated Rollback on Failure

If verification fails:
```bash
# Auto-rollback if post-hook fails
openclaw optimize --type=docs --post-hook .optimizer/verify.sh docs/
# If verify.sh exits non-zero, automatic rollback triggers

# Manual trigger
openclaw optimize --rollback docs/getting-started.md --reason "Verification failed"
```

## Dependencies

### System Requirements
- Python 3.9+
- 512MB RAM minimum (1GB+ for large batches)
- Internet connection for API calls (or local model endpoint)

### Python Packages
```txt
openai>=1.0.0        # OpenAI GPT-4
anthropic>=0.8.0    # Anthropic Claude
tiktoken>=0.5.0     # Token counting
python-dotenv>=1.0.0 # Environment management
rich>=13.0.0        # Terminal formatting
click>=8.0.0        # CLI framework
jsonschema>=4.0.0   # Config validation
gitpython>=3.1.0    # Git integration
```

**Installation:**
```bash
pip install openclaw-super-optimizer
# Or from source
git clone https://github.com/openclaw/super-optimizer
cd super-optimizer && pip install -e .
```

### API Keys (Environment Variables)

```bash
# OpenAI (required if using OpenAI provider)
export OPENAI_API_KEY="sk-..."

# Anthropic (required if using Anthropic provider)
export ANTHROPIC_API_KEY="sk-ant-..."

# Optional: Custom endpoint for local models
export OPENCLOW_OPTIMIZER_BASE_URL="http://localhost:8000/v1"

# Optional: Cost limit (USD)
export OPENCLOW_OPTIMIZER_MAX_COST="10.00"

# Optional: Default provider
export OPENCLOW_OPTIMIZER_DEFAULT_PROVIDER="anthropic"
```

Create `.env` in project root or `~/.config/openclaw/.env`.

### Local Model Support

Run with Ollama (no API cost):
```bash
ollama serve &
openclaw optimize --provider=ollama --model=llama3.2:3b draft.txt
```

Supported local backends:
- Ollama (`--provider=ollama`)
- LM Studio (`--provider=localai --base-url=http://localhost:1234`)
- Text Generation WebUI (`--provider=textgen`)

## Troubleshooting

### Common Issues

**1. "No API key found"**
```bash
# Fix: Set environment variable
export OPENAI_API_KEY="your-key-here"

# Or use config file
openclaw config set provider.openai.key "sk-..."
```

**2. "File too large, chunking failed"**
```bash
# Increase chunk size limit
openclaw optimize --max-chunk-size=50000 large_file.txt

# Or process with smaller chunks
openclaw optimize --chunk-strategy=sentence large_file.txt
```

**3. "Code block was modified" (safety error)**
```bash
# Check if AI misunderstood boundaries
openclaw optimize --debug --diff file.md | grep -A5 -B5 '\`\`\`'

# Add explicit language tags to fenced code blocks
# Before: ```` some code ````
# After: ````python some code ````
```

**4. "Rate limit exceeded"**
```bash
# Implement exponential backoff (automatic, but adjust)
openclaw optimize --retries=5 --retry-delay=2 file.md

# Switch provider temporarily
openclaw optimize --provider=anthropic file.md

# Reduce parallel workers
openclaw optimize --parallel=1 batch/
```

**5. "Optimization made text worse"**
```bash
# Use conservative mode
openclaw optimize --conservative file.md

# Reduce temperature in config
echo '{ "temperature": 0.1 }' > .optimizer/config.json

# Provide better examples in prompt
openclaw optimize --example=good_example.md --example=bad_example.md file.md
```

**6. "Git integration failed"**
```bash
# Ensure file tracked by git
git add file.md

# Force optimize without git staging
openclaw optimize --no-git file.md

# Initialize git repo if needed
git init && git add . && git commit -m "initial"
```

**7. "Memory exhausted"**
```bash
# Process files one at a time
openclaw optimize --parallel=1 large_file.txt

# Reduce model context size (cheaper model)
openclaw optimize --model=gpt-4o-mini large_file.txt
```

### Debug Mode

Enable verbose logging:
```bash
OPENCLOW_OPTIMIZER_LOG_LEVEL=DEBUG openclaw optimize file.md
# or
openclaw optimize --log-level=DEBUG file.md 2>&1 | tee debug.log
```

Debug output includes:
- Prompt sent to AI (sans API key)
- Token counts and costs per chunk
- Raw AI response before parsing
- Validation checks performed
- Git commands executed

### Getting Help

```bash
# General help
openclaw optimize --help

# Provider-specific
openclaw config show

# Validate configuration
openclaw config validate

# Test API connection
openclaw doctor --test-api

# Report bug
openclaw bug report --include-log optimize.log
```

### Performance Tuning

For large batches (>100 files):
```bash
# Measure throughput
time openclaw optimize --parallel=8 --type=docs docs/**/*.md

# Optimal parallel workers = CPU cores * 2, but respect API rate limits
# OpenAI: 3000 RPM for Tier 1, 10000 for Tier 2
# Calculate: workers = (rate_limit / avg_tokens_per_file) * safety_factor(0.8)

# Monitor real-time
openclaw optimize --parallel=4 --monitor docs/
# Shows: files/s, tokens/s, cost/s, ETA
```

## Configuration

### Config File Locations (in order of precedence):
1. `--config <path>` flag
2. `.optimizer/config.json` in current directory
3. `~/.config/openclaw/optimizer/config.json`
4. System default: `/etc/openclaw/optimizer/config.json`

### Example Config (`.optimizer/config.json`):
```json
{
  "default": {
    "provider": "anthropic",
    "model": "claude-3-5-sonnet-20241022",
    "temperature": 0.2,
    "max_tokens": 4000,
    "chunk_size": 8000,
    "max_parallel": 4,
    "backup": true,
    "git_stage": true
  },
  "profiles": {
    "docs": {
      "type": "docs",
      "style": "google",
      "rules": {
        "max_sentence_length": 30,
        "avoid_passive": true,
        "use_second_person": true
      }
    },
    "code": {
      "type": "code",
      "conservative": true,
      "preserve_comments": ["TODO", "FIXME", "HACK", "XXX"]
    }
  },
  "rules": {
    "skip_files": [".*test.*", ".*spec.*", "node_modules/"],
    "skip_patterns": ["^\\s*#", "^\\s*//\\s*\\w+:"],
    "glossary": {
      "FlickClaw": "FlickClaw",
      "OpenClaw": "OpenClaw",
      "API": "API"
    }
  },
  "cost_limits": {
    "max_per_file_usd": 0.05,
    "max_daily_usd": 10.00,
    "alert_threshold_usd": 5.00
  }
}
```

### Environment-Based Config

```bash
# Override via environment (useful for CI/CD)
export OPENCLOW_OPTIMIZER_MODEL="gpt-4o-mini"
export OPENCLOW_OPTIMIZER_MAX_PARALLEL="8"
export OPENCLOW_OPTIMIZER_NO_BACKUP="true"

# Run with overrides
openclaw optimize --type=docs docs/
```

## Advanced Usage

### Custom Prompts

Create custom prompt template at `.optimizer/prompts/<type>.jinja2`:
```jinja2
You are a {{ style }} documentation expert.

Task: Optimize the following {{ type }} content.

{{ original }}

Requirements:
{% for rule in rules %}
- {{ rule }}
{% endfor %}

Glossary (preserve these terms exactly):
{% for term in glossary %}
- {{ term }}
{% endfor %}

Output ONLY the optimized content. Do not add explanations.
```

### piping Workflow

```bash
# Integrate into pre-commit hook
cat file.md | openclaw optimize --type=docs --dry-run --diff

# If satisfied, auto-apply
cat file.md | openclaw optimize --type=docs --stdout > optimized.md && mv optimized.md file.md

# Chain with other tools
git diff --name-only '*.md' | xargs -r openclaw optimize --type=docs --git-stage
```

### CI/CD Integration

GitHub Actions (`.github/workflows/optimize-docs.yml`):
```yaml
name: Optimize Documentation
on:
  push:
    paths:
      - 'docs/**/*.md'

jobs:
  optimize:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          token: ${{ secrets.OPTIMIZER_TOKEN }}
      - name: Optimize docs
        run: |
          pip install openclaw-super-optimizer
          openclaw optimize --type=docs --style=google --git-stage docs/
      - name: Create PR
        run: |
          git config user.name 'optimizer-bot'
          git config user.email 'bot@openclaw.io'
          git checkout -b optimize/docs-$(date +%s)
          git push origin HEAD
          gh pr create --fill
```

### Integration with Editors

**VS Code** (tasks.json):
```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Optimize Document",
      "type": "shell",
      "command": "openclaw optimize --type=docs --diff ${file}",
      "presentation": {
        "echo": true,
        "reveal": "always"
      }
    }
  ]
}
```

**Neovim** (lua):
```lua
vim.api.nvim_create_user_command('Optimize', function(opts)
  local file = opts.args or vim.fn.expand('%')
  vim.fn.system(string.format('openclaw optimize --type=docs --diff %s', file))
  vim.cmd('cgetexpr system("openclaw optimize --type=docs --diff " . expand("%"))')
  vim.cmd('copen')
end, { nargs = '?' })
```

## Performance Metrics

### Benchmarks (claude-3-5-sonnet-20241022)

| File Type | Size | Time (single) | Time (4-par) | Tokens | Cost |
|-----------|------|---------------|--------------|---------|------|
| Markdown doc | 10KB | 3.2s | 1.1s | 2,500 | $0.0015 |
| Code file | 5KB | 2.1s | 0.8s | 1,800 | $0.0011 |
| Blog post | 50KB | 18.5s | 5.2s | 12,000 | $0.0072 |
| Technical spec | 100KB | 42.3s | 11.8s | 25,000 | $0.0150 |

**Throughput**: ~2,500 tokens/sec (aggregate) with 4 workers.

### Cost Optimization Strategies

1. **Use `--conservative`**: 30% fewer tokens as less rewriting
2. **Batch small files**: Combine multiple small files into single API call
3. **Cache results**: Use `--cache` (Redis/Memcached) for unchanged files
4. **Local models**: For routine optimization, use `--provider=ollama` with `llama3.2:3b`
5. **Smart skipping**: Files with no grammar issues skip AI call entirely

```bash
# Smart batch with caching
openclaw optimize --type=docs --cache-redis redis://localhost:6379 --cache-ttl=86400 docs/
```

### Monitoring

Export metrics in Prometheus format:
```bash
openclaw optimize --metrics-port=9090 --metrics-path=/metrics &
curl http://localhost:9090/metrics
# Output:
# openclaw_optimizations_total{type="docs"} 1523
# openclaw_tokens_total{provider="anthropic"} 3456789
# openclaw_cost_total_usd 12.34
# openclaw_duration_seconds_bucket{le="1"} 45
```

## Support

For issues not covered here:
- Documentation: https://docs.openclaw.io/optimizer
- GitHub Issues: https://github.com/openclaw/super-optimizer/issues
- Discord: #optimizer-support on OpenClaw Discord
- Email: support@openclaw.io

Include in bug report:
1. Command with full flags
2. Input file (or sanitized excerpt)
3. Output (if any)
4. `openclaw --version` output
5. Debug log: `OPENCLOW_OPTIMIZER_LOG_LEVEL=DEBUG ... 2>&1 | tee optim.log`

```