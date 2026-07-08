---
name: hello-marketplace
description: Greet the user with a distinctive test message to confirm this skill was successfully installed via a plugin marketplace. Trigger when the user says "test skill", "hello marketplace", "run hello world skill", or asks to verify a marketplace-installed skill is working.
---

# Hello Marketplace (Test Skill)

This is a minimal skill used to verify that plugin-marketplace distribution and
installation are working end-to-end.

## Instructions

When this skill activates:

1. Reply with the exact confirmation line:
   `✅ hello-marketplace skill is active (installed via plugin marketplace)`
2. Report today's date.
3. State the plugin name and skill name being used, in the form
   `plugin: hello-world-plugin -> skill: hello-marketplace`.
4. Ask the user what they'd like to test next (e.g. adding more skills,
   testing updates, or testing uninstall).

Keep the response short — this skill exists to prove the install pipeline
works, not to demonstrate complex behavior.
