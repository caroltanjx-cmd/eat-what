# Eat What?

A shared poll for deciding where to eat. Each person picks up to two of ten cuisines and sees everyone's picks live. The owner can reset all votes.

Live app: https://claude.ai/artifact/NSLM4PQDtcn6UDKU8wJ5UR

## How it runs

`index.html` is a single page with no build step. It runs as a **claude.ai artifact**. Login and storage come from the artifact runtime (`window.claude`):

- `user`: identifies the signed-in claude.ai viewer, with their name and avatar.
- `db`: a shared document store. Each vote is saved at `votes/<userId>` as `{ picks: [...], at }`.

The page declares these runtime capabilities when it is published:

```json
{
  "db": { "rules": [
    { "path": "votes", "read": "view", "write": "owner" },
    { "path": "votes/{self}", "write": "interact" }
  ] },
  "user": { "scopes": ["profile"] }
}
```

These rules let everyone read all the votes. Each person can write only their own vote, and only the owner can delete other people's votes (the reset).

The rules control who can write, not what they write: the db has no schema validation. The two-pick limit is enforced only in the page, so someone could write anything to their own `votes/<userId>` document. To keep that from skewing or breaking the poll, the page treats every vote as untrusted when it reads it. It ignores a `picks` value that isn't an array, drops unknown cuisine ids and duplicates, and counts at most two picks per person.

If you open `index.html` directly in a browser, or host it on GitHub Pages, the runtime isn't there. The page then shows the cuisines but asks you to sign in, and you can't vote.
