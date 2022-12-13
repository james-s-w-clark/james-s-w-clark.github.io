`hugo server` to start

To update https://github.com/paulmartins/hugo-digital-garden-theme:
```shell
git submodule update --remote --rebase
``` 
The `digital-garden` folder will change hash - can see changelog by comparing hashes against https://github.com/paulmartins/hugo-digital-garden-theme/commits/main.

--- 

# How utteranc.es was added

Pretty much follow https://mscipio.github.io/post/utterances-comment-engine/, with some butchering of a hugo theme. May make submodule updates later on tricky, but utteranc.es seems to work locally now!