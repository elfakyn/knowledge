# Fast Knowledge

No bullshit documentation. Only what you need, copy paste snippets, and the like, with short explanations.

This will hopefully reduce the incidence of spaghetti factories.

## You use "spaghetti factory" a lot. What's it mean?

It's a series of circumstances that makes it exceedingly easy to fuck up with catastrophic results. A spaghetti factory invites human error and even tricks humans into making mistakes.

Real-life examples include:
* Being allowed to just push to the master branch of a monorepo.
* NPM's "Add account to organization" feature used to look like a search box. People thought they were searching for individuals in our NPM org but all they were doing was adding "joh" "john" and "johnny" to the org, with no confirmation dialog.
* Whatever system Samsung had that made it so a random engineer could accidentally push notification "1 1" to LITERALLY EVERY PHONE IN THE UNIVERSE.
* A command that's destructive by default with a `-d` flag (meaning "nondestructive") insead of being nondestructive by default with a `--destructive` flag.
