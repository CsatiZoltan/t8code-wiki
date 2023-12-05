These are step-by-step instructions to make a new release of t8code.
These steps should only be carried out by a t8code owner (i.e. Johannes Holke).

- [ ] Ensure that all required contributions have been added to main

- [ ] Update the CITATION file

  - [ ] version number

  - [ ] release date

- [ ] Update the version number

  - [ ] Create a new branch `new_version_XYZ`

  - [ ] Edit major and minor version in the t8code version test files

  - [ ] Make a new git tag vX.Y.Z with `git tag -a vX.Y.Z` and add `This is version X.Y.Z of t8code` as description. 

  - [ ] Push the git tag

  - [ ] Create a pull request for this branch and merge it into main.

- [ ] Ensure that these changes are in the main branch and the tests pass.

- [ ] Go to the main github page of the repo and click "draft a new release"

- [ ] Write a short release note of all the (important) changes.

- [ ] Add the github generated release notes afterwards.

- [ ] Update the tarball

  - [ ] Checkout the new release in your local repository

  - [ ] Build it with `make dist`

  - [ ] Check that the tarball was build correctly

  - [ ] Unpack the tarball, configure and build t8code like a standard user would do.

  - [ ] Upload the tarball to the github release

- [ ] Add the doxygen documentation to the homepage (a corresponding PR in `t8code-website` is automatically created by T8ddy)

- [ ] Add the new version to the [docker build configuration](https://github.com/DLR-AMR/t8code-docker-images/blob/main/.github/workflows/docker-image.yml)

- [ ] Post an update article on the homepage

- [ ] Tell the DLR-SC PR Team about the release and let them post to Twitter and LinkedIn - at best with a nice picture

- [ ] ??? (add new steps, for example homepage texts)
