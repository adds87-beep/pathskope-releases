# PathSkope

A small always-on-top card for macOS that shows what the Netskope client is
doing, and switches it between tenants without reinstalling it.

The guide, with diagrams: https://adds87-beep.github.io/pathskope-releases/

Download the current build from the Releases page. Open the DMG, drag
PathSkope to Applications, launch it from there. It is signed and notarised,
so macOS opens it without a warning.

## What it shows

- **SWG**: the tunnel to the steering data plane, the POP, the gateway pool,
  the assigned address and the round trip time the client measured.
- **NPA**: the tunnel to the gateway POP, and every private app you reach:
  the host requested, the address handed back, the app and policy matched,
  the publisher.
- **GSLB**: every data plane the client measured, sorted by round trip, with
  the connected one marked, and a graph of each plane's history.

Everything comes from the client's own logs. PathSkope asks for no
permissions and captures nothing from the screen.

## Switching tenants

Register a tenant once, in Settings, Tenants: tenant name, org key, email,
auth token, and the encryption token if the tenant uses one. PathSkope
enrols against the tenant to prove the details work, then stores them
encrypted. From then on, Tools, Switch Netskope Tenant, pick the tenant,
Switch. The client is stopped, given the new tenant's files, restarted, and
the card reports each tunnel as it comes up.

- **Switch** enrols as the user saved with that tenant.
- **Switch, sign in via IdP** (Settings, Tenants) gives the client the tenant
  and no user, so the tenant's identity provider asks whoever is at the
  keyboard to sign in.
- **Unenrol the Client** (last item under Tools) backs up and removes the
  client's enrolment, so it restarts with no tenant and asks for one.
- **Stop switch** ends a switch that is going wrong. **Restore previous
  tenant** and **Enrol again from fresh** are offered when one has.

## What is stored, and how

Under `~/Library/Application Support/PathSkope/<tenant>/Data/`: the auth
token, the encryption token, the org key, the tenant's branding file and the
user certificate, each encrypted with a key that only this Mac's Secure
Enclave can unwrap. The enclave refuses without Touch ID or the login
password, so a copied folder is unreadable anywhere, including on this Mac
without you. The key is dropped after every job, after two minutes, and when
the Mac locks or sleeps. `tenants.json` beside it holds only what the menu
needs before you have touched Touch ID: tenant names, hosts, the saved email
and dates.

PathSkope's own log is at `~/Library/Logs/PathSkope/pathskope.log`. It never
contains a token, a key or a certificate.

## The helper

The steps that need root, writing the client's tenant files, restarting it,
pinning a data plane, shaping traffic, run through one small helper script
installed once with your administrator password. It accepts a fixed set of
verbs, checks every argument, and cannot be used as a general root shell.
Settings, Helper shows the installed version and, when a newer one ships,
lets you try it and roll back.

## Updates

Once a day PathSkope reads its own release list on GitHub. When a newer
recommended build is published it says so once, and Settings, Updates lets
you install it: the download is checked against this app's own signature
before it replaces the copy in Applications and relaunches. Beta builds are
offered only if you turn that on. Previous versions are listed there too, so
you can go back.

## Manufactured failover, DC pinning

Right-click a data plane in the GSLB view to add latency or fail it, so the
client fails over while you watch. Tools, DC Pinning pins the client to a POP
or country. Both are released automatically after a while and when
PathSkope quits.
