# Hexo Blog Publisher Instructions

Use these steps when you need to create, edit, or publish a new blog post for this site.

## Workflow

### 1. Create a New Post
Run the Hexo command to create a scaffold for a new post.
```bash
hexo new post "Your Post Title"
```

### 2. Edit the Post
Navigate to the newly created markdown file in `source/_posts/` and:
- Add a **Category** and **Tags** to the YAML front matter.
- Add the `<!-- more -->` tag after the introduction paragraph for the "read more" functionality.
- Implement the content of the blog post.

### 3. Generate the Site
Run the Hexo generator to build the static files.
```bash
hexo generate
```

### 4. Commit and Push Source
Commit the changes in the root directory and push them to the source repository.
```bash
git add .
git commit -m "creating new blog post: <Post Title>"
git push
```

### 5. Deploy to Public
Change directory into `public/`, which is a separate git repository for the deployed site, and push the changes.
```bash
cd public
git add .
git commit -m "Deploy: <Post Title>"
git push
```

## Agent Instructions
- Always ensure you are in the root directory of this project before running Hexo commands.
- When committing, use the descriptive message "creating new blog post: <Post Title>" for the source repo.
- Ensure the `public/` directory is updated and pushed correctly after generation.
