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

## API tokens

`atl` works with either kind of Atlassian API token.

- **Classic token.** One token for Jira and Confluence, with all the permissions of your account. It is simple
  to set up, but writes and reads use the same token.
- **Scoped tokens (recommended).** One token per app, with only the scopes `atl` needs. You can keep reads and
  writes apart with up to four tokens: Jira read, Jira write, Confluence read and Confluence write.
  Scoped tokens go through the `api.atlassian.com` gateway. `atl auth login` looks up your site's cloud id for that.

### Create a scoped token

1. Open <https://id.atlassian.com/manage-profile/security/api-tokens>.
2. Select **Create API token with scopes**.
3. Enter a name, for example `atl jira read`, and an expiry date (1 to 365 days).
4. Select the app: **Jira** or **Confluence**.
5. Select the scopes from the table below for that token.
6. Select **Create**, then copy the token. It starts with `ATATT`. Atlassian shows it only once.

Do this once for each token you want.

| Token | Scopes | Used by |
|-------|--------|---------|
| Jira read | `read:jira-work`, `read:jira-user` | `get`, `search`, `find`, `comments`, `transitions`, `editmeta`, `user`, `--assignee me`, and the current values that `edit`, `edit-comment` and `transition` show before they write |
| Jira write | `write:jira-work` | `edit`, `create`, `comment`, `edit-comment`, `transition` |
| Confluence read | `read:page:confluence`, `read:space:confluence`, `search:confluence` | `get`, `search`, `find`, `auth status`, and the version check and diff before `update` |
| Confluence write | `write:page:confluence`, `write:comment:confluence` | `update`, `comment` |

Notes:

- The Jira scopes are on the **Classic** scopes tab. The Confluence page and space scopes are on the
  **Granular** tab, and `search:confluence` is on the **Classic** tab. If the form will not combine them,
  use the granular `read:content-details:confluence` in place of `search:confluence`.
- A write token needs no read scopes, because `atl` reads with the read token. `atl auth set-token` tests a
  token with a read call, so a write-only token fails that test. Answer `y` to save it anyway.
- If you need only Jira, skip the Confluence tokens, and the other way round.
- A token can do only what your account can already do. Scopes limit a token. They never add permissions.

### Log in

```sh
atl auth login                        # site, email, then classic or scoped tokens (the read tokens)
atl auth set-token jira-write         # optional, scoped tokens only
atl auth set-token confluence-write
atl auth status
```

With scoped tokens, `atl` sends no writes to an app until you add its write token. With a classic token, the
same token does reads and writes. To replace an expired token, run `atl auth set-token jira` (or `confluence`,
`jira-write`, `confluence-write`).

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
