![tailwindcss logo](.\img\tailwindwss-logo.png)

# Tailwind Cheat Sheet

## Table of Contents

[Setup](#setup)

[Text Size](#text size)

[Responsive](#responsive)

[Flexbox](#flexbox)

## Setup

+ Create a **node.js** project in a folder : `npm init -y`

+ Then install **tailwindcss** using npm : `npm install -D tailwindcss`

+ Create tailwindcss conf file using : `npx tailwindcss init`

+ Define the template path inside `tailwind.config.js`
  
  ```javascript
  content: ['./src/**/*.{html,js}']
  ```

+ Create the `src` and `public` folders

+ Inside the `src` folder create `styles.css`

+ We import the basic functionalities of tailwindcss inside it
  
  ```css
  @tailwind base;
  @tailwind components;
  @tailwind utilities;
  ```

+ Inside `package.json` add a script shortcut to compile `style.css` to the `public` folder
  
  ```javascript
  "scripts": {
      "build-css": "tailwindcss -i ./src/styles.css -o ./public/styles.css --watch"
  }
  ```

+ To compile the css,   you have to run : `npm run build-css`

## Responsive

## Flexbox
