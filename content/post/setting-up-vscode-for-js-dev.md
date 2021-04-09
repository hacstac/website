---
title: "Setting Up Visual Studio Code For Front_End_Development!!"
date: 2019-09-20T20:00:00+08:00
tags: ["Productivity", "Programming"]
categories: ["Productivity"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### Why Visual Studio Code ?? 💻

---

#### What is VS_Code? & Why is it so Popular?

Visual Studio Code (Vs-Code) is a source code editor developed by Microsoft that can be run on all major OS's in the world (Windows, Mac-OS, and Linux). It is free, open-source, and provides support for debugging as well as built-in Git version control, syntax highlights, snippets, and so on. The UI of Vs-code is highly customizable, as users can switch to different themes, keyboard shortcuts, and preferences.

<!--more-->

Vs-code has initially been announced in 2015 as an open-source project hosted on GitHub before releasing to the web a year later. Since then, Microsoft's code editor has been gaining popularity among developers.

In the Stack Overflow 2018 Developer Survey, Vs-code was ranked as the most popular development environment with around 35% out of over 100,000 respondents on the basis of its usage. More stunningly, this figure is about 39% in the web development field.

And with monthly updates, users can expect to enjoy an even better experience — bug fixes, stability, and performance boosts are frequently pushed.

___

## Themes :

- Color Themes
- Icon Themes


### Color Themes

- [Cobalt2 Theme](https://github.com/wesbos/cobalt2-vscode) {{ Best Theme }}

![Cobalt2-Theme](https://cdn.hashnode.com/res/hashnode/image/upload/v1569003333698/-A3O8MZp3.png)

This theme requires some configuration when you will open the above link then it will show some recommended settings which we are covered in our setup, so you don't need to configure separately if you have followed this article. But we need to set up CSS hacks for extra smooth editor this will also require some configuration , so we will first install plugin called [ vscode-custom-css](http://be5invis/vscode-custom-css) after installing the plugin, you will need to create a file `.vscodestyles.css` in your root directory `Windows - C:/Users/${USER}/AppData/Roaming/Code/User/.vscodestyles.css` and copy the code from [cobalt2-custom-hacks.css](https://github.com/wesbos/cobalt2-vscode/blob/master/cobalt2-custom-hacks.css) and then go to your editor enable custom CSS `Ctrl + Shift + P ➢ Search Enable Custom CSS and JS` and paste your file location in settings.json in vs code which is given in configuration file below.

- [Fairy Floss](https://github.com/sailorhg/fairyfloss) {{ Purple Background Theme }}

![Fairy Floss](https://cdn.hashnode.com/res/hashnode/image/upload/v1569003819054/jhmfNapq4.png)

- [Dracula Official](https://draculatheme.com/) {{ Dark Background Theme }}

![Dracula](https://cdn.hashnode.com/res/hashnode/image/upload/v1569003930835/eUHT-Og80.png)

- [Night Owl](https://github.com/sdras/night-owl-vscode-theme) {{ Theme for Night Owls ( Fine-tuned for those of us who like to code late into the night) }}

![NightOwl](https://cdn.hashnode.com/res/hashnode/image/upload/v1569003970965/OkwkQZjEO.png)

- [Monokai Pro](https://www.monokai.pro/) {{ Dark Brown Background Based Theme}}

> But some developers recommend that instead of this Theme, You should install One Monokai Theme Because [One Monokai](https://vscodethemes.com/e/azemoh.one-monokai) is a cross between Monokai and one dark theme. Monokai Pro.

![MonokaiPro](https://cdn.hashnode.com/res/hashnode/image/upload/v1569004029913/7bgDXsmqb.png)

- [Oceanic Next](https://marketplace.visualstudio.com/items?itemName=naumovs.theme-oceanicnext)
Oceanic Next

![OcenicNext](https://cdn.hashnode.com/res/hashnode/image/upload/v1569004073345/5ZHWygPXj.png)

___

### File Icon Themes

- [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)

![MaterialIcon](https://cdn.hashnode.com/res/hashnode/image/upload/v1569004262606/zSqYdTqIs.png)

- [VS-Code Icons](https://marketplace.visualstudio.com/items?itemName=vscode-icons-team.vscode-icons)

![VsCodeIcons](https://cdn.hashnode.com/res/hashnode/image/upload/v1569004288874/-Aejwpac2.jpeg)


___

## Fonts

Install "fontExample.ttf" file in your system and set "Selected_Font" in Editor

- [FiraCode iScript](https://github.com/kencrocken/FiraCodeiScript)

![FiraCodeIScript](https://cdn.hashnode.com/res/hashnode/image/upload/v1569004515875/yJxdzBbr6.png)

- [FiraCode](https://github.com/tonsky/FiraCode)

![FiraCode](https://cdn.hashnode.com/res/hashnode/image/upload/v1569004544756/CgEtMQmas.jpeg)

- [Operator-mono-lig](https://github.com/kiliman/operator-mono-lig)

![OperatorMono-lig](https://cdn.hashnode.com/res/hashnode/image/upload/v1569004585308/0Dd9KXFRt.png)

- [Firicico](https://github.com/kosimst/Firicico)

![Firicico](https://cdn.hashnode.com/res/hashnode/image/upload/v1569004618945/e6kiZqpEg.png)

> {{ My Font for Setup : FiraCode iScript }}

___

## Configuration of Editor

Visual Studio allows also us to edit the underlying Settings.json config file.
File ➢ Preferences ➢ Settings ➢ Then click the "Curly Brackets" to open "Settings.json

- Editor Settings
- WorkSpace Settings
- Other Configurations

### Editor Settings :

1. Set Font Size = 16, Line Height = 25, Font Weight = 400, Settings Tab_Size = Two Spaces, Cursor Width = 5, Cursor_Blinking = Solid, Cursor_Style = Block, Letter Spacing = 0.5, Rulers = 100 , 100 .
2. Font_Ligatures = In a programming language, we are use many operators and symbols, but still, they represent a single symbol, so vs code support font ligatures and can be used to improve the rendering of source code on the screen.
3. Format_on_Save = To save individual files, use the new save without formatting command, which is easier than globally turning on and off format on save.
4. Format_on_Paste = It will preserve format on copying.
5. Minimap = Shows overview of your code on right hand side of screen.
6. Rounded_Selection = Apply rounded borders to selection.
6. Auto_Closing_Brackets = Auto close the brackets.
7. Auto_Indent = Editor should automatically adjust the indentation when user will type, paste or move line.
8. Scroll_Beyond_Lastline = This will prevent the editor from scrolling beyond the last line.
9. Word_Wrap = Lines should be wrapped {{ Not exceed to the screen }}, so you don't need to scroll right with this.
10. Match_Brackets = Highlight the matching brackets, when one of them is selected.
11. Render_WhiteSpace = Show white space characters.
12. Smooth_Scrolling = Scroll using an animation.
13. Show_Folding_Controls = Show folding option to code ( -,+).
14. Light_Bulb = Disable code action light bulb in the editor.
15. Quick_Suggestion_Delay = When quick suggestions will show up, it will delay in milliseconds.
16. Snippet_Suggestion = Code snippets are templates that make it easier to enter repeating code patterns.
17. Color_Decorators = Editor should render the inline color decorator and color picker.
18. Parameter_Hints = Disable the popup that shows parameter documentation and type information as we type.
19. Token_Color_Customizations = Overides editor and font style from the currently selected color theme.


### Workbench Settings :

1. Statusbar_Visible = Visibility of the status bar at the bottom of the workbench.
2. Show_Tabs = Opened editors should show in tabs or not.


### Other Configurations :

This includes terminal settings, files settings, and plugins settings.

- Note: In Window Title you need to copy your favorite Emoji {remember to Delete brackets around it } & paste , these emojis shown with your file name .

- In terminal shell select [zsh](https://ohmyz.sh/) if you have already installed [zsh](https://ohmyz.sh/) terminal, otherwise select bash. Window.Title Settings

![HeaderImage](https://cdn.hashnode.com/res/hashnode/image/upload/v1569005899838/oSIF2_f9g.png)


___

## Extensions :

1. Auto Close Tag {{ Automatically add HTML/XML close tag, same as Visual Studio IDE or Sublime Text does. }}
2. Auto Rename Tag {{ Automatically rename paired HTML/XML tag, same as Visual Studio IDE does. }}
3. Babel JavaScript {{ JavaScript syntax highlighting for ES201x, React JSX, Flow and GraphQL. }}
4. Beautify {{ Beautify JavaScript, JSON, CSS, Sass, and HTML in Visual Studio Code. }}
5. Better Comments {{ The Better Comments extension will help you create more human-friendly comments in your code. With this extension, you will be able to categories your annotations into: Alerts, Queries, To Dos, Highlights, Commented out code can also be styled to make it clear the code shouldn't be there, Any other comment styles you'd like can be specified in the settings. }}
6. Bracket Pair Colorizer {{ This extension allows matching brackets to be identified with colours. The user can define which characters to match, and which colours to use. }}
7. Carbon-now-sh {{ A VS Code extension to open the current editor content in [carbon.now.sh](https://carbon.now.sh/ "https://carbon.now.sh"). }}
8. Change-case {{ A wrapper around [node-change-case](https://github.com/blakeembrey/node-change-case "https://github.com/blakeembrey/node-change-case") for Visual Studio Code. Quickly change the case of the current selection or current word. }}
9. Check Updates of NPM Packages {{ Visual Studio Code extensions that check if all packages in dependencies and dev-dependencies sections of your package.json files are up to date. }}
10. Color Highlight {{ This extension styles css/web colors found in your document. }}
11. Color Picker {{ Helper with GUI to generate color codes such as CSS color notations. And, a command Convert Color to change the color notation. }}
12. Css Formatter {{ Adds Formatting to CSS. }}
13. Custom Css and Js Loader {{ Use in Cobalt2 Theme. }}
14. Debugger for Chrome {{ A VS Code extension to debug your JavaScript code in the Google Chrome browser, or other targets that support the [Chrome DevTools Protocol](https://chromedevtools.github.io/debugger-protocol-viewer/ "https://chromedevtools.github.io/debugger-protocol-viewer/"). }}
15. Document This {{ "Document This" is a Visual Studio Code extension that automatically generates detailed JSDoc comments for both TypeScript and JavaScript files. }}
16. DotENV {{ A port of [DotENV](https://github.com/zaynali53/DotENV "https://github.com/zaynali53/DotENV") for vscode. }}
17. Duplicate Selection {{ This extension adds an action to vscode to duplicate the current selection. }}
18. EditorConfig For VS Code {{ This plugin attempts to override user/workspace settings with settings found in .editorconfig files. }}
19. ES7 React/Redux/GraphQL/React-Native/Js Snippets {{ This extension provides you JavaScript and React/Redux snippets in ES7 with Babel plugin features for [VS Code](https://code.visualstudio.com/ "https://code.visualstudio.com/") }}
20. Expand-region {{ Porting sublime-expand-region to visual code. }}
21. File Templates {{ This allows us to quickly create new files based on defined templates. }}
22. File Utils {{ A convenient way of creating, duplicating, moving, renaming, deleting files and directories. }}
23. File-Icons {{ Famous VS Code File Icons. }}
24. Find Related Files {{ Finds files related to the current file based on user-defined configuration rules. }}
25. Git History
26. hex-rgba converter {{ Color code converter for hex to rgba. }}
27. htmltagwrap {{ Wraps your selection in HTML tags. Can wrap inline selections and selections that span multiple lines. }}
28. Import-cost {{ This extension will display inline in the editor the size of the imported package. }}
29. Indent Rainbow {{ This extension colorizes the indentation in front of your text. }}
30. JavaScript Console utils {{ Easily insert and remove console.log statements }}
31. Language-Stylus {{ Adds syntax highlighting and code completion to Stylus files in Visual Studio Code. }}
32. Markdown All in one
33. MDX {{ For .md Files }}
34. Multi Line Tricks {{ This plugins enables the line seletion and end line multi cursor behaviour of Sublime Text to VSCode. }}
35. Now {{ now.sh Implementation }}
36. Npm {{ This extension supports running npm scripts defined in the package.json file and validating the installed modules against the dependencies defined in the package.json. }}
37. Npm Intellisense
38. Open-in-Browser {{ Preview html file in your browser, firefox & google chrome & IE. }}
39. Paste & Indent
40. Path AutoComplete {{ Provides path completion for visual studio code. }}
41. Path Intellisense {{ AutoComplete Filenames }}
42. Polacode {{ You have spent countless hours finding the perfect [JavaScript grammar](https://marketplace.visualstudio.com/search?term=javascript%20grammar&target=VSCode&category=All%20categories&sortBy=Relevance "https://marketplace.visualstudio.com/search?term=javascript%20grammar&target=VSCode&category=All%20categories&sortBy=Relevance"), matching it with a [sleek-looking VS Code theme](https://marketplace.visualstudio.com/search?target=VSCode&category=Themes&sortBy=Downloads "https://marketplace.visualstudio.com/search?target=VSCode&category=Themes&sortBy=Downloads"), trying out all the [best programming fonts](https://www.slant.co/topics/67/~best-programming-fonts "https://www.slant.co/topics/67/~best-programming-fonts"). }}
43. PostCSS Syntax {{ Provide Syntax highlighting for PostCss files. }}
44. Prettier — Code Formatter {{ VS Code package to format your JavaScript / TypeScript / CSS using [Prettier](https://github.com/prettier/prettier "https://github.com/prettier/prettier"). }}
45. Project Manager {{ Easily Switch between Projects }}
46. Quokka.js {{ Live ScratchPad for Js }}
47. Setting Sync {{ Recommanded }}
48. saveBackup {{ Backup File when you save }}
49. Toggle Quotes
50. Turbo Console Log {{ This extension make debugging much easier by automating the operation of writing meaningful log message. }}
51. Vandelay JavaScript
52. Version Lens {{ Shows package version information for npm, jspm, bower, dub and dotnet core in the [Visual Studio Code](https://github.com/microsoft/vscode "https://github.com/microsoft/vscode") editor. }}
53. View In Browser
54. VS-Code HackerTyper {{ Great for live coding presentations, impressing your friends, or just trying to look busy at work, Hacker Typer allow us to record Your self programming }}
55. vscode-styled-components {{ Syntax highlighting and IntelliSense for [styled-components](https://github.com/styled-components/styled-components "https://github.com/styled-components/styled-components"). }}
56. WakaTime {{ Metrics, insights, and time tracking automatically generated from your programming activity. }}
57. Wallaby.js {{ For Testing Javascript Code }}

___

## VS-Code Icons 😈

- [dhanishgajjar/vscode-icons](https://github.com/dhanishgajjar/vscode-icons)


![FileIcon.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1569007445142/D3Be5rMlb.gif)


___
