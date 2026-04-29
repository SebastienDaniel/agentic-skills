
# Code Style

## import / export
* **Never** export unused types, interfaces, constants or functions. Excessive exports kill the encapsulation value of modules.

## Comments

* Explain "why" not "what" - avoid obvious/redundant comments
* If code structure (names, control flow) communicates intent, no comment needed
* For complex logic blocks (10+ lines), use delimiter comments for "what"
* Comments describe current state, not evolution - that is what versioning tools and PRDs (project requirement documents) are for
* Error handling is descriptive and actionable, it requires no additional comments

## No Section Delimiters

Don't mark sections with comments ("// Test Helpers", "// Public Methods", ASCII art). File structure should be self-evident. If markers seem needed, split the file.
