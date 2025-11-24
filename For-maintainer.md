# Guidance for maintainer

## Set auto-update for Submodule

1. What can auto-update function do?
   The auto-update function is used for auto-committing the submodule to the top repo. The file structure in out lab looks like:

   ```
   - PowerXLab (top repo)
        - HTML
        - CSS
        - JS
        - index.html
        - Resources (submodule)
            - icons
            - images
            - ...
        - Guidance (submodule)
            - Examples
            - xxx.md
    ```

    It means when someone adds or deletes something in the submodule repo, the top repo will be auto updated.

2. Generate a ***Fine-grained tokens***
   
   2.1. Click your profiles -> Settings -> Developer settings -> Personal access tokens -> Fine-grained tokens. You need to choose the organization ***power-x-lab-web***, and select the top repo ***PowerXLab***.

   ![PAT setting 1](./Images/PAT_Key_Setting_1.png)

Add ***contents read and write*** permissions.

![PAT setting 2](./Images/PAT_Key_Setting_2.png)

![PAT setting 3](./Images/PAT_Key_Setting_3.png)

3. Go to the submodule ->  settings -> Secrets and Variables -> Actions -> New Repository Secrets.

![submodule action](./Images/SubmoduleAction.png)

Name the secret as ***SUPER_REPO_TOKEN***, and paste the PAT key.

![New secret](./Images/SubmoduleActionSecrets.png)

4. Create a new file in the root folder of the submodule `.github/workflows/bump-super.yml`

```
name: Bump super repo submodule

on:
  push:
    branches: [ master ]  # 或者你自己的默认分支

jobs:
  bump-super:
    runs-on: ubuntu-latest

    steps:
      # 1. 检出顶层 repo
      - name: Checkout super repo
        uses: actions/checkout@v4
        with:
          repository: power-x-lab-web/PowerXLab   # ✅ 顶层仓库全名
          token: ${{ secrets.SUPER_REPO_TOKEN }}
          submodules: true                       # 把 submodule 一起拉下来

      # 2. 更新 submodule 指到最新提交
      - name: Update submodule to latest
        run: |
          cd ./         # ✅ 顶层 repo 中 submodule 的路径
          git fetch origin master         # ✅ submodule 默认分支
          git checkout origin/master
          cd -

          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

          git add path/to/submodule
          git commit -m "chore: bump submodule to latest" || echo "No changes to commit"
          git push origin master          # ✅ 顶层 repo 默认分支
```