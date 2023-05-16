# use Chirpy Starter   
[jekyll-theme-chirpy](https://rubygems.org/gems/jekyll-theme-chirpy)    
[getting-started](https://chirpy.cotes.page/posts/getting-started/)

# start (docker)
```
// local env
sudo docker run -it --rm \
    --volume="$PWD:/srv/jekyll" \
    -p 4000:4000 jekyll/jekyll \
    jekyll serve
```