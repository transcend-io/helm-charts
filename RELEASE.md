This chart releases automatically via CI on push to the `main` branch.

# Steps required to release a new chart version manually:

1.  Create a new release branch from main branch
    ```bash
        git checkout -b <developer_name>/release_<version>
    ```
2.  Update the version in `chart.yaml` in root folder with the version to be released. 
3.  Create the new release package.
    ```bash
        helm package .
    ```
4. Update the index file
    ```bash
        helm repo index --url https://transcend-io.github.io/helm-charts/ .
    ```
5.  Push the branch after commiting the changes
6.  Request for PR review.
7.  Once PR is merged by repo owner, a new version is published by Github Pages.