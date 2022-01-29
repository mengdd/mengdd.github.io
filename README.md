# mengdd.github.io
Github pages of mengdd : http://mengdd.github.io/

## Branches
**hexo** branch is default branch, used for management, configurations and posts.

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

or you could use homebrew:
```
brew install nodejs
```

### Hexo
```
$ npm install -g hexo-cli
```

or you could use homebrew:
```
brew install hexo
```

### First clone
cd this project and run:
```
npm install
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

## Theme
Current theme: https://github.com/theme-next/hexo-theme-next

I have a fork here: https://github.com/mengdd/hexo-theme-next-1.

To update the theme:

cd `themes/next`
```
git fetch upstream
git merge upstream/master
```
Fix conflicts and commit.
