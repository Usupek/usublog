---
title: My Blog Pipeline
date: 2024-12-12
draft: false
tags:
  - blog
  - scripting
---
## The Setup

In this blog I will show you a walkthrough of how my Blog Pipeline works. and maybe you can follow along!

but first, let's get the setup done.

*disclaimer: I will only show a walkthrough for windows OS*
## Obsidian
If you haven't downloaded obsidian, go download it, it's awesome: https://obsidian.md/

### Write post
after Obsidian is installed, write something in the folder you want to use (for me it's posts folder). 

when you wanna write something and upload it to your blog, **always** have some **frontmatter**. Like this for example:

``` bash
---
title: My Blog Pipeline
date: 2024-12-12
draft: false
tags:
  - blog
  - scripting
---
```

put this on the top of your obsidian file that you wanna upload to your blog.

After you're done writing, then look for the folder you use for your post's directory (This will be needed later). Like this for example:
!![Image Description](/images/Pasted%20image%2020241212193541.png) ^2e1ce9

## Hugo

As for the site I use Hugo.
> A quick intermezzo about Hugo:
> **Hugo** is an open-source static site generator written in Go. It **generates HTML** pages from source content written in markdown and can be used **to build websites, blogs, and other types of web content**.


### Install Hugo
 to install Hugo just go to this site: https://gohugo.io/installation/

*an important thing to note: You'll need to have **Git**, and **Go** installed in your system*

### Create a new site
after installing Hugo, go ahead and **create a new site**
``` bash
hugo version ## verify hugo works

## create a new site
hugo new site website_name
cd website_name
```

### Hugo themes
after creating a new site, find Hugo Themes that you like: https://themes.gohugo.io/

follow the theme instructions on how to download. The **Best** option is to install as a **git module.**

``` bash
## inititalize a git repo (in Hugo website directory)
git init

## install a theme
git submodule add -f https://github.com/example-theme.git theme/terminal
```

after the theme is installed, you will have to **adjust the Hugo settings**. *Most themes* will have an example configuration you can use. This is usually **the best way** to make sure Hugo works well and out of the box.

edit the *hugo.toml* to make the changes. like me for example, I use hello-friend-ng theme and this is the configuration the theme gave:
``` bash
baseurl      = "localhost"
title        = "My Blog"
languageCode = "en-us"
theme        = "hello-friend-ng"
paginate     = 10

[params]
  dateform        = "Jan 2, 2006"
  dateformShort   = "Jan 2"
  dateformNum     = "2006-01-02"
  dateformNumTime = "2006-01-02 15:04"

  # Subtitle for home
  homeSubtitle = "A simple and beautiful blog"

  # Set disableReadOtherPosts to true in order to hide the links to other posts.
  disableReadOtherPosts = false

  # Enable sharing buttons, if you like
  enableSharingButtons = true
  
  # Show a global language switcher in the navigation bar
  enableGlobalLanguageMenu = true

  # Metadata mostly used in document's head
  description = "My new homepage or blog"
  keywords = "homepage, blog"
  images = [""]

[taxonomies]
    category = "blog"
    tag      = "tags"
    series   = "series"

[languages]
  [languages.en]
    title = "Hello Friend NG"
    keywords = ""
    copyright = '<a href="https://creativecommons.org/licenses/by-nc/4.0/" target="_blank" rel="noopener">CC BY-NC 4.0</a>'
    readOtherPosts = "Read other posts"

  [languages.en.params]
    subtitle  = "A simple theme for Hugo"

    [languages.en.params.logo]
      logoText = "hello friend ng"
      logoHomeLink = "/"
    # or
    #
    # path = "/img/your-example-logo.svg"
    # alt = "Your example logo alt text"

  # And you can even create generic menu
  [[menu.main]]
    identifier = "blog"
    name       = "Blog"
    url        = "/posts"
```

you can modify it if you want.

### Run server
now test run the server
``` bash
hugo server -t themename
```

## Walkthrough
after we're done with the setup, we can start doing the fun part, **The Walkthrough**.

## Syncing Obsidian to Hugo

first we have to sync the post we want to upload to Hugo.

``` bash
robocopy source_path destination_path /mir
```

what does this do is basically copy our **markdown file** (file that we wanna upload) to Hugo's content folder

*change the source_path to your posts folder as shown in [here](#^2e1ce9).*

*change the destination_path to Hugo /content folder*

### Handling images
There's a slight issue on handling images from Obsidian to Hugo. The images in Obsidian is not saved **on** the same folder your obsidian post file. So to get over it, I use this script make by **Network Chuck:**

**images.py**:
``` python
import os
import re
import shutil

# Paths (using raw strings to handle Windows backslashes correctly)
posts_dir = r"C:\Users\Rafi\Documents\usublog\content\posts"
attachments_dir = r"C:\Users\Rafi\Documents\Obsidian Vault\attachments"
static_images_dir = r"C:\Users\Rafi\Documents\usublog\static\images"

# Step 1: Process each markdown file in the posts directory
for filename in os.listdir(posts_dir):
    if filename.endswith(".md"):
        filepath = os.path.join(posts_dir, filename)
        
        with open(filepath, "r", encoding="utf-8") as file:
            content = file.read()
        
        # Step 2: Find all image links in the format ![Image Description](/images/Pasted%20image%20...%20.png)
        images = re.findall(r'\[\[([^]]*\.png)\]\]', content)
        
        # Step 3: Replace image links and ensure URLs are correctly formatted
        for image in images:
            # Prepare the Markdown-compatible link with %20 replacing spaces
            markdown_image = f"![Image Description](/images/{image.replace(' ', '%20')})"
            content = content.replace(f"[[{image}]]", markdown_image)
            
            # Step 4: Copy the image to the Hugo static/images directory if it exists
            image_source = os.path.join(attachments_dir, image)
            if os.path.exists(image_source):
                shutil.copy(image_source, static_images_dir)

        # Step 5: Write the updated content back to the markdown file
        with open(filepath, "w", encoding="utf-8") as file:
            file.write(content)

print("Markdown files processed and images copied successfully.")

```

## Push to Github

after you've done all that, it's basically done. now you just have to push it to your **Github repo**.

*make sure you're in your server directory.* Then type this to your terminal to push to your **Github repo**:

``` bash
## add remote repo if haven't
git remote add origin git@github.com:User/UserRepo.git

## add all changes in current directory to workspace
git add .

## commit changes
git commit -m "example commit message"

## push the commit to remote repo's master branch
git push -u origin master
```

alright!! all done, now you can choose to **deploy** it to your platform of choice. I use **netlify** for example.

## Mega script!
here's the mega script to make things easier:

``` bash
# PowerShell Script for Windows

# Set variables for Obsidian to Hugo copy
$sourcePath = "C:\Users\Rafi\Documents\Obsidian Vault\posts"
$destinationPath = "C:\Users\Rafi\Documents\usublog\content\posts"

# Set Github repo 
$myrepo = "git@github.com:Usupek/usublog.git"

# Set error handling
$ErrorActionPreference = "Stop"
Set-StrictMode -Version Latest

# Change to the script's directory
$ScriptDir = Split-Path -Parent $MyInvocation.MyCommand.Definition
Set-Location $ScriptDir

# Check for required commands
$requiredCommands = @('git', 'hugo')

# Check for Python command (python or python3)
if (Get-Command 'python' -ErrorAction SilentlyContinue) {
    $pythonCommand = 'python'
} elseif (Get-Command 'python3' -ErrorAction SilentlyContinue) {
    $pythonCommand = 'python3'
} else {
    Write-Error "Python is not installed or not in PATH."
    exit 1
}

foreach ($cmd in $requiredCommands) {
    if (-not (Get-Command $cmd -ErrorAction SilentlyContinue)) {
        Write-Error "$cmd is not installed or not in PATH."
        exit 1
    }
}

# Step 1: Check if Git is initialized, and initialize if necessary
if (-not (Test-Path ".git")) {
    Write-Host "Initializing Git repository..."
    git init
    git remote add origin $myrepo
} else {
    Write-Host "Git repository already initialized."
    $remotes = git remote
    if (-not ($remotes -contains 'origin')) {
        Write-Host "Adding remote origin..."
        git remote add origin $myrepo
    }
}

# Step 2: Sync posts from Obsidian to Hugo content folder using Robocopy
Write-Host "Syncing posts from Obsidian..."

if (-not (Test-Path $sourcePath)) {
    Write-Error "Source path does not exist: $sourcePath"
    exit 1
}

if (-not (Test-Path $destinationPath)) {
    Write-Error "Destination path does not exist: $destinationPath"
    exit 1
}

# Use Robocopy to mirror the directories
$robocopyOptions = @('/MIR', '/Z', '/W:5', '/R:3')
$robocopyResult = robocopy $sourcePath $destinationPath @robocopyOptions

if ($LASTEXITCODE -ge 8) {
    Write-Error "Robocopy failed with exit code $LASTEXITCODE"
    exit 1
}

# Step 3: Process Markdown files with Python script to handle image links
Write-Host "Processing image links in Markdown files..."
if (-not (Test-Path "images.py")) {
    Write-Error "Python script images.py not found."
    exit 1
}

# Execute the Python script
try {
    & $pythonCommand images.py
} catch {
    Write-Error "Failed to process image links."
    exit 1
}

# Step 4: Build the Hugo site
Write-Host "Building the Hugo site..."
try {
    hugo
} catch {
    Write-Error "Hugo build failed."
    exit 1
}

# Step 5: Add changes to Git
Write-Host "Staging changes for Git..."
$hasChanges = (git status --porcelain) -ne ""
if (-not $hasChanges) {
    Write-Host "No changes to stage."
} else {
    git add .
}

# Step 6: Commit changes with a dynamic message
$commitMessage = "New Blog Post on $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')"
$hasStagedChanges = (git diff --cached --name-only) -ne ""
if (-not $hasStagedChanges) {
    Write-Host "No changes to commit."
} else {
    Write-Host "Committing changes..."
    git commit -m "$commitMessage"
}

# Step 7: Push all changes to the main branch
Write-Host "Deploying to GitHub Master..."
try {
    git push origin master
} catch {
    Write-Error "Failed to push to Master branch."
    exit 1
}

Write-Host "All done! Site synced, processed, committed, built, and deployed."
```
