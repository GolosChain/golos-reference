Name blinding
-------------

- This is `theoreticalbts` idea for an interesting feature

This is a feature implemented in Namecoin.  It is a commit/reveal procedure to prevent front-running of name registration.
When registering a new name, you can *commit* `(H(name + separator + salt), recipient_pubkey)` in one tx, then within 24 hours,
*reveal* `salt` in another tx to claim the name.  If multiple claims to the same name are submitted, the claim with the
*earliest commit time* is given priority.  NB the recipient pubkey is given in the commit, not the reveal, so someone else
front-running your reveal pays a fee but doesn't gain the name.

Note, this can result in situations where account name is revoked (because it tried to claim a name that was revealed earlier).
So the named object (e.g. account, but are account objects the only named objects in Graphene?) still exists, but just becomes
nameless.