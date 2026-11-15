Uli's Notes
===========

Intro
-----

I'm not a GO developer and I do have some memory issues.
So I'll have to keep some
notes in case I have to rebase my work.

File "go.sum"
-------------

Don't care too much on solving merge conflicts for it.

- Merge it "somehow"
- Re-create "go.sum" afterwards by:
  ```
  rm go.sum
  go mod download
  go mod tidy
  ```

Build
-----

Just run `./build.bash`.
