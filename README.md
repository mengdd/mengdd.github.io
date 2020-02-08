# mengdd.github.io
Github pages of mengdd : http://mengdd.github.io/


## branches
**hexo** branch is for management, configurations and posts.

**master** branch is for static files, which is generated and deployed using Hexo.

## Environments
See [Hexo docs](https://hexo.io/docs/).

Need Git, Node.js and Hexo.

### Node.js
First nvm:
```
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
```
Then restart the terminal and:
```
$ nvm install stable

```

### Hexo
```
$ npm install -g hexo-cli
```

## Commands
See [Hexo commands](https://hexo.io/docs/commands.html).

### Create new post
```
$ hexo new [layout] <title>
```
The posts are in 'source/_posts' folder.

### Start a local server
```
$ hexo server
```
By default at: http://localhost:4000/

### Generate and deploay
```
$ hexo g -d  //or hexo d -g
```

## Markdown
image:
```
![alt text](/images/image-name.png)
```


