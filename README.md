# atl

A small command line client for Jira Cloud and Confluence Cloud. It is one Python file and needs only the
standard library.

- **Reads freely, writes carefully.** Every write prints a preview or a diff and sends nothing until you add `--yes`.
- **Separate read and write tokens.** Reads never use a write token. A `POST` that is not a confirmed write is blocked.
- **Tokens stay in the macOS Keychain.** The config file holds only the site, the email and options.
- **Markdown in, Markdown out.** Issue descriptions and comments are written in Markdown and converted to the
  Atlassian Document Format, with support for headings, nested lists, tables, code blocks, quotes, links and
  `@[Name](accountid:ID)` mentions.
- **Safe Confluence updates.** `--base-version` stops an update when someone else changed the page after you read it.

## Requirements

- Python 3.9 or later
- macOS, because tokens are stored with the `security` Keychain tool

## Install

```sh
git clone https://github.com/gusleig/atl-cli.git
ln -s "$PWD/atl-cli/atl" ~/.local/bin/atl   # or any directory on your PATH
```

## Log in

Create API tokens at <https://id.atlassian.com/manage-profile/security/api-tokens>, then run:

```sh
atl auth login                    # site, email, classic or scoped tokens
atl auth set-token jira-write     # optional: a separate token for writes
atl auth set-token confluence-write
atl auth status
```

## Usage

```sh
atl jira get PROJ-123 --comments
atl jira find "billing export" --project PROJ --open
atl jira search 'assignee = currentUser() AND statusCategory != Done'
atl jira edit PROJ-123 --points 3 --description-file desc.md          # dry run
atl jira edit PROJ-123 --points 3 --description-file desc.md --yes    # sends
atl jira comment PROJ-123 --file note.md --yes
atl jira transition PROJ-123 "In Progress" --yes

atl confluence find "runbook" --space ENG
atl confluence get https://your-company.atlassian.net/wiki/spaces/ENG/pages/12345/Title
atl confluence update 12345 --file page.html --base-version 7 --yes
```

Run `atl --help` or read the docstring at the top of `atl` for every command.

## Config

`atl auth login` writes `~/.config/atl/config.json` with mode 600. You can add these optional keys:

| Key | Default | Purpose |
|-----|---------|---------|
| `default_project` | none | The project for `atl jira find` when you pass no `--project` |
| `points_field` | `customfield_10014` | The Story Points field id on your site |
| `sprint_field` | `customfield_10010` | The Sprint field id on your site |

Custom field ids differ from site to site. `atl jira editmeta PROJ-123` lists the ids on your site.

## License

MIT
