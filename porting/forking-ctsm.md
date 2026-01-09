# **CTSM Forking and Submodule Management Guide**

This guide provides a step-by-step process for forking the **CTSM** repository, managing its dependent repositories (submodules), and modifying the repository structure for development.

---

## **1. Forking the CTSM Repository**
1. Navigate to the CTSM repository on GitHub:
   **[https://github.com/ESCOMP/CTSM](https://github.com/ESCOMP/CTSM)**
2. Click the **"Fork"** button in the upper right to create your copy under your GitHub account.
3. Ensure that **all branches are copied**, not just the `master` branch.
4. Clone your fork locally using SSH (recommended for convenience):
   ```bash
   git clone git@github.com:your-username/CTSM.git
   cd CTSM
   ```
   If you prefer HTTPS, use:
   ```bash
   git clone https://github.com/your-username/CTSM.git
   ```

---

## **2. Setting Up the Upstream Remote**
After cloning your fork, add the original CTSM repository as an upstream remote:
```bash
git remote add upstream https://github.com/ESCOMP/CTSM.git
```
Verify the remotes:
```bash
git remote -v
```
You should see output similar to:
```
origin  git@github.com:your-username/CTSM.git (fetch)
origin  git@github.com:your-username/CTSM.git (push)
upstream  https://github.com/ESCOMP/CTSM.git (fetch)
upstream  https://github.com/ESCOMP/CTSM.git (push)
```
- **origin** → Your fork (read/write access).
- **upstream** → The original CTSM repository (read-only access unless you have permissions).

To update your fork with the latest changes from upstream:
```bash
git fetch upstream
git merge upstream/main  # Replace 'main' with the correct branch
```

---

## **3. Checking Out a Specific Branch**
To work on the latest CTSM release, switch to the appropriate branch:
```bash
git checkout ctsm5.3.023
```
Create a new branch for modifications:
```bash
git checkout -b uf-ctsm5.3
```

---

## **4. Managing Git Submodules**
CTSM includes submodules (nested repositories), such as `ccs_config`, which we need for the config files and which must be managed properly.

### **4.1. Forking and Updating Submodules**
We need to fork and modify `ccs_config`:
1. Fork **[https://github.com/ESMCI/ccs_config_cesm](https://github.com/ESMCI/ccs_config_cesm)**.

2. Clone your fork locally:
   ```bash
   git clone git@github.com:your-username/ccs_config_cesm.git
   cd ccs_config_cesm
   ```

3. Find the tag used in your __CTSM__ checkout. You can find this by looking at the `.gitmodules` file in `$CTSMROOT`.

   ```ini
   [submodule "ccs_config"]
   path = ccs_config
   url = https://github.com/ESMCI/ccs_config_cesm.git
   fxtag = ccs_config_cesm1.0.20
   fxrequired = ToplevelRequired
   # Standard Fork to compare to with "git fleximod test" to ensure personal forks aren't committed
   fxDONOTUSEurl = https://github.com/ESMCI/ccs_config_cesm.git
   ```

4. Create a new branch to work on.

   ```bash
   git checkout ccs_config_cesm1.0.20
   git checkout -b uf-config
   ```

5. Add the HiPerGator config files to your cloned repo. Commit and push the changes.

6. Create a tag that can be added to `.gitmodules`

   ```bash
   # create tag and push to github
   git tag -a uf-config1.0 -m "Tagging version v1.0 of uf configs"
   git push origin uf-config1.0
   ```

7. Update the `fxtag`  line in`.gitmodules` to point to your forked submodule:

   ```ini
   [submodule "ccs_config"]
   fxtag = uf-config1.0
   ```

### **4.2. Running `git-fleximod` to Update Submodules**
After modifying submodules, run:
```bash
./bin/git-fleximod update
```
Commit the updated submodule reference:
```bash
git add ccs_config
git commit -m "Updated submodule reference after git-fleximod update"
git push origin uf-ctsm5.3
```

---

## **5. Creating and Managing Git Tags**
If you need to create a new tag:
```bash
git tag -a tag-v1.0 -m "Tagging version v1.0"
```
Push the tag:
```bash
git push origin tag-v1.0
```
### **Overwriting an Existing Tag**
If you need to update an existing tag:
```bash
git tag -d tag-v1.0
```
Delete the old tag from the remote:
```bash
git push --delete origin tag-v1.0
```
Create a new tag with the same name:
```bash
git tag -a tag-v1.0 -m "Updated tag"
git push origin tag-v1.0
```
### **Deleting Multiple Local Tags**
To delete multiple local tags:
```bash
git tag -d tag1 tag2 tag3
```
To delete the same tags remotely:
```bash
git push --delete origin tag1 tag2 tag3
```
To refresh tags from the remote repository:
```bash
git fetch --prune --tags
```

---

## **6. Summary of Workflow**
1. **Fork CTSM and its submodules**, ensuring all branches are copied.
2. **Clone your fork using SSH** for convenience.
3. **Add the original CTSM repository as `upstream`** and fetch updates regularly.
4. **Checkout the appropriate branch and create a working branch for modifications.**
5. **Manage submodules properly** by forking them and updating `.gitmodules`.
6. **Run `git-fleximod update` to update submodules.**
7. **Create, update, or delete Git tags as needed.**

