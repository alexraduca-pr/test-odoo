# AGENTS.md

Project: custom Odoo 19 modules deployed on Odoo.sh.

This file is for AI agents working in this repository. Keep changes simple,
Odoo-native, and easy for the next developer to review.

## 1. Context

- Target version: Odoo 19.0. Check each module's `__manifest__.py`; do not
  assume older Odoo 17 or 18 APIs.
- Deployment: Odoo.sh. Each Git branch corresponds to an Odoo.sh environment:
  main for production, develop for staging, and other branches for test or feature work.
- Runtime: Python 3.12+ and PostgreSQL 16+.
- Odoo core and standard addons are read-only. Extend behavior through
  inheritance instead of editing Odoo source.

## 2. Repository Shape

Typical custom module layout:

```text
module_name/
  __manifest__.py
  __init__.py
  models/
  views/
  security/
  data/
  demo/
  tests/
```

## 3. Source Language and Naming

- Write code, comments, docstrings, commit messages, XML labels, and field
  strings in English.
- Keep source strings in English and translate user-facing text with `.po`
  files when needed.
- Use descriptive Odoo names:
    - Models: `module_name.model_name`, class name in `CamelCase`.
    - Fields: `snake_case`; Many2one fields end with `_id`; One2many and
      Many2many fields end with `_ids`; booleans usually start with `is_` or
      `has_`.
    - Methods: `snake_case`; private helpers start with `_`; button methods
      start with `action_`; computed, onchange, and constraint methods start
      with `_compute_`, `_onchange_`, and `_check_`.
    - XML record IDs: `<module>_<model_or_area>_<purpose>`.
- Prefer one model per Python file when adding new models.

## 4. Security

- Every persistent new model needs an access rule in
  `security/ir.model.access.csv`, unless it is abstract or transient and that
  exception is clear from the model.
- Add record rules when data is company-specific, user-specific, or otherwise
  restricted.
- Use `sudo()` only for a documented permission boundary, such as reading
  system configuration. Add a short comment explaining why elevated access is
  required.
- Ask before changing existing security files unless the user explicitly
  requested the security change.

## 5. Views and Assets

- Inherit views with explicit `xpath expr=` selectors.
- Avoid replacing top-level view nodes unless there is no safer extension point.
- Keep JavaScript and OWL code in `static/src/`; do not place inline scripts in
  XML views.
- Keep XML labels and `string` attributes in English.

## 6. Test Case Creation

### File organization

Organize test files by business process or risk area, not by Odoo model.
- If a test involves several models linked by a causal flow, put it in a single file describing that
  flow, even if it touches 3 different models.
- Create a file dedicated to a single model only when that model has custom
  logic complex enough to not depend on other models (e.g. an isolated
  `@api.constrains`, a compute with internal branching).
- Name files after the tested behavior, `test_<process>.py`, never after the
  model.
- Place each test in the module that owns the behavior it exercises, and only
  import fixtures/helpers from a module listed in the current module's
  `depends`. Never import from a module that depends on the current one, or
  from an unrelated module outside the dependency chain.
- Each module's test suite must be runnable on its own, without requiring a
  downstream module to be installed.

### What to test

Use risk-based coverage: test what can break silently, not everything that
exists. Do not write a test per method; write a test per identified risk
(corrupted data, wrong calculations, inconsistent state, unhandled
exceptions).

Before writing a test, state the risk it covers in a one-line comment above
the test method. If you cannot articulate a concrete risk, do not write the
test. Before adding any test, ask: "if I remove this test, do I lose the
ability to detect a real bug a user or process would hit?" If the answer is
no, omit the test.

Write a test when the code:
- Overrides or fights core Odoo behavior (e.g. a `compute` that calls
  `super()` and then changes the result, an overridden `create()`/`write()`).
- Adds a guard that protects existing data or enforces a state machine
  (raises `UserError`/`ValidationError`); cover both the blocked and the
  allowed path.
- Computes a non-obvious value: magic constants, UoM/unit conversions,
  aggregations, or a subtle condition (e.g. `"x" not in vals` vs
  `vals.get("x")`).
- Propagates a change between related records (e.g. archiving a parent
  cascades to its children).

For `@api.constrains`, write at most one test for the failing case plus one
for the valid case, unless the method has genuinely branched logic (several
independent invalid conditions).

Group assertions about the same scenario into a single test method instead
of one method per assert.

Don't write a test for:
- Plain `_inherits` delegation (checking that the delegated record exists).
- Native ORM/field behavior you did not override: a plain `Many2one`
  assignment, an optional field, a default `ondelete`.
- A simple `active`/archive toggle with no propagation and no guard.
- `message_post`/chatter side effects.
- Smart button action dictionaries (`res_model`, `domain`), unless the domain
  itself is built from non-trivial logic.
- Standard ORM CRUD, `Char`/`Boolean`/`Selection` fields without custom
  logic, XML views, translations, or security already covered by a standard
  `ir.model.access.csv` entry.
- A pure delegation/wrapper method with no logic of its own.

## 7. Translation Workflow

- Translation files live in each module's `i18n/*.po` (e.g. `i18n/it.po`).
- Only translate user-visible strings: field labels, help texts shown in the
  UI, view labels, selection values, button/menu names, and error messages
  raised with `_()`. Do not translate developer-facing text such as code
  comments, log messages, technical field/model names, or docstrings.
- If a `.po` file ends up with entries whose `msgstr` stays empty (no
  translation provided), remove those entries from the file instead of
  leaving them blank.
- Keep the `#:` location comments as generated; do not hand-edit them.
- Regenerate `.po` files from Odoo's export (Settings > Translations >
  Import/Export, or `odoo-bin --i18n-export`) rather than hand-crafting
  `msgid` entries, so they stay in sync with the source strings.

## 8. Git Workflow

- Keep one logical task per commit.
- Use Odoo-style commit summaries:

```text
[TAG] module: short imperative summary
```

- Common tags: `[ADD]`, `[FIX]`, `[IMP]`, `[REF]`, `[REM]`, `[REV]`, `[MOV]`,
  `[I18N]`.
- Keep the first line in English, imperative, and at most 72 characters.
- Before committing, verify `git config user.name` and `git config user.email`.
  If either is unset, ask the user.
- Do not push directly to `main`, `master`, `production`.
- From the Odoo.sh editor, use `odoosh-push`. From a normal local clone,
  `git push` is acceptable.

## 9. Ask Before

Ask for confirmation before:

- Changing manifest dependencies, license, or major version.
- Creating or editing migrations.
- Changing security access or record rules, unless explicitly requested.
- Deleting or renaming existing fields that may contain production data.
- Modifying a module marked stable or frozen in its own documentation.

## 10. References

- [Odoo 19 documentation](https://www.odoo.com/documentation/19.0/)
- [Odoo 19 ORM API](https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html)
- [Odoo 19 ORM changelog](https://www.odoo.com/documentation/19.0/developer/reference/backend/orm/changelog.html)
- [Odoo 19 testing](https://www.odoo.com/documentation/19.0/developer/reference/backend/testing.html)
- [Odoo 19 module manifests](https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html)
- [Odoo upgrade guidelines](https://www.odoo.com/documentation/19.0/developer/reference/upgrades.html)
- [Odoo.sh documentation](https://www.odoo.com/documentation/19.0/administration/odoo_sh.html)
- [Odoo coding guidelines](https://www.odoo.com/documentation/19.0/contributing/development/coding_guidelines.html)
- [Odoo Git guidelines](https://www.odoo.com/documentation/19.0/contributing/development/git_guidelines.html)

For anything not covered here, follow the linked Odoo 19 documentation and the
existing style of the module being changed.

Always link the origin of your responses for reference.
