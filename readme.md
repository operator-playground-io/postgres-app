Go to github.com, create a public repository
Unzip the downloaded contents on your workstation
You will see a folder contribution, all the contents inside the folder contribution should be hosted on the public git repository created as part of step 1
Note the commit Id(full sha commit Id) of the commit after you upload the contents from contribution folder to git repository
Copy Git url(Https)
Update the file stacks.json in the git repository with the value of Git url and GIT commit id
Clone the repository https://github.com/i-data-explorer/data-explorer-catalog Data Explorer Catalog
Put your stacks.json file contents in _files/stacks.json of cloned repository.(Do verify that the stack_id is unique)
Raise Pull Request
Wait for apporval
Once merged, access the Dashboard application and see the contributed asset in the catalog