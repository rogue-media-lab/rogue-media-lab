This article is about setting up two github accounts on your computer. This way I can work on the studio and personal things from the same system.
- LIVE

# Don’t Shoot Yourself in the Foot: Git, GitHub, and the Two-Account SSH Fix
### Let's set up a shiny new Github and config some ssh keys for two separate accounts
date: oct 09, 2025

I have no idea who is reading this, but I do know one thing, if you are playing with code, (whether you’re vibing with AI or you’re a seasoned developer) you will kick yourself if you aren’t familiar with Git.

Git is a version control system built by the guy who created Linux, Linus Torvalds. It allows many people to work on the same file at the same time. It is pretty awesome. You might be thinking you don’t need it because you’re the only one who touches your projects. I’m here to tell you that you’re missing out. Even if you code solo, using Git is a professional necessity that ensures your future self is your best collaborator.

Think back to the old days of sharing a word document. One person worked on it, saved it, and sent it over. If several people opened it simultaneously, everyone had their own local version. The last one to save was the version you got, and it might look nothing like what you worked on.

With Git, those sections of the file that don’t match are flagged as merge conflicts and need to be reviewed before merging. There can be only one final version, which ensures the file is the best of everyone’s work, not just the last person to touch it.

Git also offers branches. One branch might contain the stable version, while another can be used to build a radically different new feature. This is often how new features are built and tested safely. Git isn’t the only version control system, but I’ve never run across a developer who hasn’t heard of it.

Getting Started: Git & GitHub Essentials
GitHub is version control in the cloud, and it is awesome. Computer crash? Your work is safe on GitHub. Found an open-source project like Rails you want to contribute to? It’s on GitHub. If you don’t have an account, stop reading and go grab one.

Assuming you have Git installed on your system (most everything does today), here are the first commands you should run:

Check your version:

git --version
Check your global credentials. These are used to sign your commits, so they should match the name and email you use for GitHub:

git config --list --global
If you need to set them, do it now. This will save you a ton of headaches later.

git config --global user.name “Your Name”
git config --global user.email “Your Email”
The Scenario: Moving a Multi-Branch Repo
I’m reorganizing a few things, and I need to update and move some projects. I have a repository (repo) on my personal GitHub that includes some Rails 8 templates. I need to pull it down, fix things up, and move it to a brand-new GitHub account. The repo has several branches, each with a different template, perfect for a ride-along!

For whatever reason, I can’t find the original project on my laptop. No problem, I have the repo on GitHub! I grab the SSH code URL to clone the repo (SSH is recommended by GitHub for security) and run:

git clone git@github.com:Developer3027/rails-8-templates.git
If I cd into the new folder and run git status, I see everything is clean:

On branch main
Your branch is up to date with ‘origin/main’.

nothing to commit, working tree clean
But wait, where are all my branches? Running git branch --list only shows the local branch, main. To see the other branches from the repo, I need the remote flag:

git branch -r
Now I see all the remote branches:

  origin/HEAD -> origin/main
  origin/flash-fade
  origin/flowbite
  origin/main
  origin/tailwind
  origin/tailwind-8
If I want to start working on the tailwind branch, I run git switch tailwind. You can also use git checkout, but switch is the newer, dedicated command for changing branches, whereas checkout is overloaded with other functions (like restoring files). Pick your poison, they both work.

Awesome, I have the original and can work it!

Share Rogue Media Lab

The Deep Dive: Configuring Multi-Account SSH
Now, let’s look at the new GitHub account. It’s called Rogue Media Lab and is brand new. So shiny.

I need to fix the SSH key issue. I already had two keys, but I opted to make some organizational changes that will make things cleaner moving forward (though it means a minor fix for my other projects).

If you need to create an SSH key, check the GitHub documentation. For me, running Linux, I know the folder holding my keys is hidden at ~/.ssh. Here’s what my directory originally looked like:

masonroberts@pop-os:~/.ssh$ ls
id_ed25519.pub  id_ed25519  gitlab.pub  id_gitlab
I knew the id_ed25519 key was for my personal GitHub account. I opened the gitlab.pub file, copied its contents, and pasted it as a new SSH key into the Rogue Media Lab GitHub settings. Done!

To test it, I created a new repo called rails-templates and cloned it. I copied some files from the old repo into the new local folder, ran a git add . and a git commit -m ‘initial’.

However, when I ran git push origin main I got a message back saying that Developer3027 can’t do that. The issue was which key the system was trying to use. Running ssh -T git@github.com still responded with “Hi Developer3027”, meaning it was using my personal key.

The fix was to configure the system to use one key for my dev account and one for the Rogue account. I renamed the two private keys in my .ssh folder and then added a file called config.

Here is what the directory now looks like:

masonroberts@pop-os:~/.ssh$ ls
config      id_ed25519.pub  id_rsa_rogue  gitlab.pub  id_rsa_dev
And here is the magic within the config file:

# Developer3027 Account (Original Key/Default Connection)
Host github.com-dev
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_dev
  IdentitiesOnly yes

# Rogue Media Lab Organization Account (New Key)
Host github.com-rogue
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_rogue
  IdentitiesOnly yes

# Optional: Keep the default GitHub connection for personal repos cloned before
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_dev
  IdentitiesOnly yes
To ensure the SSH agent was aware of the renamed files, I added them:

ssh-add ~/.ssh/id_rsa_dev
ssh-add ~/.ssh/id_rsa_rogue
It turns out GitHub didn’t care that my local file names changed; the keys themselves were untouched.

Join Mason Roberts’s subscriber chat
Available in the Substack app and on web
The Final Fix: Updating the Remote URL
Here is the part I need to remember moving forward. To clone a repo, Git saves the URL as the remote. You can see this URL by running git config --list in a project and finding remote.origin.url. For SSH, it starts with git@github.com:.

To use the key we configured in the config file, we must modify the remote URL to use the corresponding Host alias. Since the rails-templates project belongs to Rogue Media Lab, I need to use the github.com-rogue alias we defined.

I did this by running the following command in my rails-templates directory:

git remote set-url origin git@github.com-rogue:rogue-media-lab/rails-templates.git
Now, if you run git remote -v, you will see both fetch and push use the correct key:

git remote -v
# origin  git@github.com-rogue:rogue-media-lab/rails-templates.git (fetch)
# origin  git@github.com-rogue:rogue-media-lab/rails-templates.git (push)
And that’s it! Now I just need to remember to change that remote alias when I switch between working on my personal repos and the Rogue Media Lab projects. When I see that message that the commit failed, I know exactly what to do.

This was an awesome learning experience for me. I’m pleased with the outcome, this new venture is now fully separated from my personal account and stands on its own. I hope this helps you if you ever run into the same issue. At the very least, I hope you enjoyed the ride-along!