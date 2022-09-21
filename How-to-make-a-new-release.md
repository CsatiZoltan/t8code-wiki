These are step-by-step instructions to make a new release of t8code.
These steps should only be carried out by a t8code owner (i.e. Johannes Holke).

- [ ] Ensure that all required contributions have been added to main

- [ ] Update the CITATION file

- - [ ] version number

- - [ ] release date

- [ ] Update the version number

- - [ ] Edit major and minor version in the t8code version test files

- - [ ] Make a new git tag vX.Y.Z with `git tag -a `

- - [ ] Push the git tag

- [ ] Ensure that these changes are in the main branch

- [ ] Go to the main github page of the repo and click "draft a new release"

- [ ] Add the doxygen documentation to the homepage (This process should get automated in future)

- - [ ] `make doxygen`

- - [ ] Goto t8code-website repo and create the folder `content/doc/vX.Y.Z`

- - [ ] Copy the content of the doxygen generate `html` folder to `content/doc/cX.Y.Z`

- - [ ] Edit the website page `content/pages/4_Documentation.md`, such that the old version appears in the list and the new version is pointed to by the 'latest' entry

- [ ] If desired, post an update article on the homepage (Definitely do this for major releases)

- [ ] ??? (add new steps, for example homepage texts)
