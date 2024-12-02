# [wiki.amdcloud.kz](https://wiki.amdcloud.kz)

> [!WARNING]
> Some of information was taken from [virtualize.link](https://virtualize.link/). \
> The idea itself was realized by referencing to [virtualize.link](https://virtualize.link/). \
> Make sure to check out the [virtualize.link](https://virtualize.link/) and star author's [repo](https://github.com/quietsy/advanced-configurations).

## Contributing the project
### Requirements
- Make sure Python is installed, updated and working (3.9 or 3.12)
- Make sure Pip is installed, updated and working
### Cloning the repository
1. Open terminal and execute commands:
```bash
git clone https://github.com/TeamDominant/Wiki
```
2. Enter the directory you just cloned
```bash
cd Wiki
```
3. Create venv, because it is required to be activated every time you interact with project
```bash
python -m venv venv
```
4. Activate venv
Linux:
```bash
source venv/bin/activate
```
Windows:
```bash
venv/Scripts/activate
```
5. Install the `mkdocs-material`
```bash
pip install mkdocs-material
```
### Publishing the page to localhost to monitor changes
Make sure venv is activated and mkdocs is installed successfully
```bash
mkdocs serve
```
If port is already in use specify the port you want to use
```bash
mkdocs serve -a localhost:63456
```
### Commiting the changes
Case 1: You are the project contributor, __not__ the member of the organization
1. Open a PR
2. Commit changes
3. Wait for the review

Case 2: You __are__ an organization member
0. Make sure GitLens is installed in VS Code
1. Commit changes via GitLens with proper comment and then Sync
2. Deploy via terminal, if the changes were accepted by Team Lead

Universal:
1. Create a draft branch and create draft PR from it
2. Apply for review your changes
