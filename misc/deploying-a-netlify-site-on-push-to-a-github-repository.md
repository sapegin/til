<!-- 2020-08-13  -->

# Deploying a Netlify site on push to a GitHub repository

Imagine, we have a project on GitHub with some documentation in the same repository, and we have a site for this project in another repository that takes the documentation for the first repository. We want every change in the project repository to trigger a site deploy. We can do this with Netlify webhooks.

1. Go to Build & Deploy settings of the site on Netlify, and under Build hooks press **Add build hook**.

2. Go to Webhooks setting of the _site_ repository on GitHub, and press **Add webhook**.

3. Put the URL from the first step, and leave all other values as defaults.





