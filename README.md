# Licence failsafe for STVN Lab - TelePrompter ("dead man's switch")

Product `teleprompter` on the STVN licence server (serials TELEPR-…). Repository name: `teleprompter-failsafe`.

This repository makes sure customers keep using what they bought if the licence server ever disappears for good.

- The licence server ([stvn-licence-server](https://github.com/stevennoulet/stvn-licence-server)) updates
  `alive.json` here once a day.
- Every day, GitHub checks it (`.github/workflows/failsafe.yml`). When it is **45 days** old, the signed unlock file
  (kept until then as the secret `FAILSAFE_UNLOCK`) is published as `unlock.lic`. Apps that look for it then treat
  every permanent licence as the unlock edition, for good, without the server.
- Customers can't trigger it, and the unlock can't be faked: it is signed with the product's private key, which only
  the licence server has.

Your app has to support this: after some days without the server it downloads `unlock.lic` from this repository
and checks its signature (see `docs/licence-format.md` in the server's repository).

## Set-up (once per product)
1. **Create a public repository** from this folder (e.g. `<your user>/<app>-failsafe`), including the hidden
   `.github` folder. It must be public: apps read `unlock.lic` without logging in.
2. **Token for the licence server**: GitHub → Settings → Developer settings → Personal access tokens →
   **Fine-grained tokens** → Generate new token → Only select repositories → this repository →
   Repository permissions → **Contents: Read and write**. Choose the longest expiry; the server e-mails you 30 days
   before it ends.
3. **Licence server settings** of the product (`product.json` "settings", or compose.yaml for the default product):
   `FAILSAFE_REPO` = `<user>/<repository>`, `FAILSAFE_TOKEN` = the token. Restart. Within a minute `alive.json`
   appears here, and the product's admin page shows "last still alive".
4. **The unlock file**: admin page → Failsafe → **Download unlock file**. Then in **this repository** (not your
   account settings): the **Settings** tab at the top of the repository page (gear icon; on a narrow window it is under
   **…**) → left sidebar, section **Security** → **Secrets and variables** → **Actions** → **New repository secret** →
   name `FAILSAFE_UNLOCK`, value = the whole file. Then delete the download.
5. **Test**: Actions → Licence failsafe → Run workflow (leave "no") → it should end with "OK".

## Good to know
- From day 30 without a heartbeat the daily run fails on purpose: GitHub e-mails you "Run failed" — a warning
  15 days before the unlock.
- Server down but you are fine: bring it back within 45 days, or edit `alive.json` here (`"unix"`: the current time
  in seconds) to push the date out.
- Emergency, by hand: Actions → Licence failsafe → Run workflow → type `YES`.
- Once `unlock.lic` is published, apps that have seen it keep it, even if you delete it again.
- GitHub turns off scheduled workflows in a repository without activity for 60 days; the server's daily update is
  that activity, and 45 days stays inside it.
- After you change the product's signing key or its edition names, download the unlock file again and replace the
  secret.
