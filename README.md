This repository contains Course Notes for 50.005, as well as Lab handouts and Programming Assignment Handouts. More content will be updated as time progresses.

To change logo:
- replace _includes/svg/logo.svg with your file named logo.svg as well
  
To add a new content, create markdown files in any of the folders:
- os_notes
- ns_notes
- assignments
- ns_labs
- os_labs

Use the front matter template as given in the sample called `overview.md`. 

Then, edit `_data/navigation.yml` to add it on the sidebar navigation. Note that the **sidebar-nav** and **nav-key** in the front matter of the new `.md` file must match the name you declare in this navigation yaml file. 

## Changing logo
Replace the .svg file inside `/_includes/svg/logo.svg` with the same name. 

## Custom CSS
Add any custom sass styling in `/_sass/custom.scss`

## Custom html for articles
Add any additional elements in `/_includes/article/footer/custom.html`. You may also add it at `/_includes/article/top/custom.html` if you want it to execute first. 
Current added features:
- Go to top button at `/_includes/article/top/custom.html`
- Collapsible answer box at `/_includes/article/footer/custom.html`
- All styling in `custom.scss`


