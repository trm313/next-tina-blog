---
published: true
title: Responsive animated sidebar layout in React and Tailwind
---

# What we’re making

In this article we will use React (create-react-app) and Tailwindcss to create a responsive layout that contains:

* Sidebar navigation on desktop, which can be minimized and maximized with an animated effect. It’ll be scroll-able, independent of content
* A bottom-nav navigation on mobile, fixed to bottom of screen
* A scrollable content container, with a header

The completed project code can be found on GitHub here: https://github.com/trm313/tut\_react\_tailwind\_sidebar

![Sidebar navigation, expanded state](/uploads/sidebar_maximized.png "Sidebar navigation, expanded state")

# On Tailwindcss

None of this requires Tailwindcss (or React) to accomplish, of course, but if you have decided to leverage Tailwind, or are thinking about it, it’s very important to establish practices that play to it’s strengths, as well as leveraging React’s power alongside it effectively.

Tailwindcss, like any styling mechanism, can get messy if used poorly - we want to be careful not to mix approaches and over-use our CSS files, while also focusing on smart abstraction of components and using loops to mitigate duplicative work. When allowed to shine, Tailwind adds serious value:

* No more thinking about class names all the time
* Not jumping between files when building or editing
* Not dealing with conflicting styles or worrying about side effects from changes
* It keeps your CSS bundle smaller

I find the utility-first style drastically speeds up my development, feels so much less tedious than working in separate CSS files, and is even easier to read (once you get the hang of it).

# Let’s get into it!

## Set up create-react-app and Tailwindcss

We’ll start by spinning up a create-react-app application:

```
npx create-react-app react_tailwind_sidebar
cd react_tailwind_sidebar
```

Then install and initialize Tailwindcss ([setup guide](https://tailwindcss.com/docs/guides/create-react-app)):

```
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Open up `tailwind.config.js` and set up Tailwind’s config to point to our `src` files. I’ve also extended the gray color pallet here with some nicer ones to work with.

In our `index.css` we will add the Tailwind directives to the top of our file

```jsx
// tailwind.config.js
module.exports = {
  content: ["./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {
      colors: {
        gray: {
          100: "#f1f1f5",
          200: "#b6bbc9",
          300: "#6c7793",
          400: "#4c5366",
          600: "#2b303b",
          900: "#1d1f24",
        },
      },
    },
  },
  plugins: [],
};
```

Tailwind establishes three layers to allow for smart, cascading styles, in order, from base → components → utilities. See [Tailwindcss Functions and Directives](https://tailwindcss.com/docs/functions-and-directives) for more information. Anything added below it will apply over the default layers.

```jsx
/* src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  //...
}

code {
  //...
}
```

We’ll add another one other thing to our `index.css`, but generally we won’t be looking to leverage stylesheets in order to play to Tailwind’s strengths:

```jsx
/* src/index.css */
// ..
body {
  // ...
  font-size: 16px;
}
// ...
```

We’re assigning `font-size: 16px` to `body` in order to establish our `rem` calculation point. Tailwind `size` properties reference `rem` units by default, versus `px` . `rem` is a relative unit, and is a good practice to use so that things like sizes and spacing stay proportional across our app. It makes it easier to manage our work, as well as helping things scale for accessibility purposes. Test it out by zooming in with `CTRL` `+` later on once we have more stuff in here.

Go ahead and boot up your app

```
npm start
```

## Get our Layout and Content set up

Create a folder `src/components` and a new file `Content.js`

Here we’re just going to add some hipster ipsum so we can see where we’d be putting the content for various routes.

Copy it from here **(**[…src/components/Content.js](https://github.com/trm313/tut_react_tailwind_sidebar/blob/main/src/components/Content.js)) or add generate some yourself. Just try to make sure its long enough to overflow the page so that we can see how our scrolling is working.

```jsx
// src/components/Content.js
export default function Content() {
  return (
    <div className="flex flex-col p-4 content">
      <p className="my-4">// lots of hipster ipsum</p>
    </div>
  );
}
```

Create a new file `src/components/Layout.js` and we’ll just get the basics set up

```jsx
// src/components/Layout.js
const Layout = ({ children }) => {
  return <div>{children}</div>;
};
export default Layout;
```

And back in `App.js` we’ll wrap Layout around our Content and get rid of all the default CRA stuff

```jsx
// src/App.js
import Content from "./components/Content";
import Layout from "./components/Layout";

function App() {
  return (
    <Layout>
      <Content />
    </Layout>
  );
}
export default App;
```

## Responsive Layout - Sidebar versus Bottom Nav

On large viewports we want to show a sidebar on the left some some navigation links, and on small viewports we want to have a horizontal navbar fixed at the bottom of the screen.

We will use flexbox and Tailwindcss media queries to create this layout.

We’ll start with some basic mock-up

```jsx
// src/components/Layout.js
const Layout = ({ children }) => {
  return (
    <div
      className={`
      h-screen w-screen overflow-hidden
      flex flex-col-reverse lg:flex-row
      lg:overflow-y-auto lg:overflow-x-hidden
      `}
    >
      <nav className="bg-gray-900 text-gray-100">
        <p>Link 1</p>
        <p>Link 2</p>
        <p>Link 3</p>
      </nav>
      <div className="flex flex-col overflow-y-auto">{children}</div>
    </div>
  );
};
export default Layout;
```

Our outmost `div` is a flex, controlling two descendants, our navigation element, and our content.

On larger screens, we want our navigation bar to render to the *left* of our content, which it will do given the ordering with our `flex-row` class.

On small screens, we apply a flex direction of `flex-col-reverse` so that our navbar and content are vertically aligned, but with our navbar *below* the content.

We’re using `h-screen` and `w-screen` to apply “100vh” and “100vw” respectively to our Layout, and by setting `overflow-hidden` we’re enabling ourselves to control scroll behavior more minutely, which we will do by setting `overflow-y-auto` on each of our `nav` element, and the `div` surrounding our `{children}` (but just for large viewports on the navbar). This will mean our sidebar and content can each scroll separately.

We’ll also add `lg:overflow-x-hidden` so there’s no chance of a pesky horizontal scrollbar on our sidebar

Now we’ll refactor that `nav` into its own component file NavBar.js and add some more responsive styling so that our navigation bar renders its elements horizontally for small screens, and vertically for large screens.

```jsx
// src/components/NavBar.js
const NavBar = () => {
  return (
    <nav
      className={`
			bg-gray-900 text-gray-100

      flex flex-row justify-around
			lg:flex-col lg:justify-start
      w-full lg:w-[18rem]

      lg:p-2
      lg:overflow-y-auto
      `}
    >
      <p>Link 1</p>
      <p>Link 2</p>
      <p>Link 3</p>
    </nav>
  );
};
export default NavBar;
```

We’re establishing a `flex-row` for small screens, and `flex-col` for large screens. On small screens we want the navigation items to span across the row evenly, which we’ll use `justify-around` for. On large screens, we want our nav items to start at the top, however, so we’ll set `lg:justify-start` to handle that behavior.

We’re also responsively setting the width to `w-full` on small screens, and `lg:w-[18rem]` for large ones.

And with our `lg:overflow-y-auto` we’ll let our sidebar content be scrollable, in case our nav items end up longer than the viewport height. This lets us have a separate scroll for our NavBar than we’ll have for the main content.

Now we’ll head into Layout, import our NavBar, and render it above the `div` containing our `{children}`

```jsx
// src/components/Layout.js
import NavBar from "./NavBar";
const Layout = ({ children }) => {
  return (
    <div
      className={`
      h-screen w-screen overflow-hidden
      flex flex-col-reverse lg:flex-row
      lg:overflow-y-auto lg:overflow-x-hidden
      `}
    >
      <NavBar />
      <div className="flex flex-col overflow-y-auto">{children}</div>
    </div>
  );
};
export default Layout;
```

## Adding routes to populate our NavItems

For our nav links in NavBar we are going to have several links that will all look the same. To keep styling clean, we can create those links in an array, and then map over the array to generate each item.

We want each of our nav items to have an icon, so we’ll use react-icons to help us grab some. From your terminal, run:

```
npm install --save react-icons
```

Create a new folder under `src` called `constants` and a file inside called `routes.js`

```jsx
// src/constants/routes.js
import { HiBookOpen, HiRectangleGroup, HiInbox } from "react-icons/hi2";
import { BsPeopleFill } from "react-icons/bs";

const routes =
  {
    text: "Inbox",
    icon: <HiInbox />,
  },
  {
    text: "Projects",
    icon: <HiRectangleGroup />,
  },
  {
    text: "Recruiting",
    icon: <BsPeopleFill />,
  },
  {
    text: "Learning",
    icon: <HiBookOpen />,
  },
];
export default routes;
```

We’ll create a new component called NavItem.js in our `src/components` folder

```jsx
// src/components/NavItem.js
const NavItem = ({ text = "Default", icon = "" }) => {
  return (
    <div className="flex-grow lg:flex-grow-0 flex flex-col lg:flex-row items-center p-4 cursor-pointer rounded-lg">
      <div className="flex justify-center items-center text-4xl text-gray-400">
        {icon}
      </div>
      <p className={`text-sm lg:text-lg lg:ml-4 text-gray-200`}>{text}</p>
    </div>
  );
};
export default NavItem;
```

And in our NavBar component, we’ll map over these routes, and render a NavItem for each:

```jsx
// src/components/NavBar.js
import routes from "../constants/routes";
import NavItem from "./NavItem";
const NavBar = () => {
  return (
    <nav
      className={`
			bg-gray-900 text-gray-100

      flex flex-row justify-around
			lg:flex-col lg:justify-start
      w-full lg:w-[18rem]

      lg:p-2
      lg:overflow-y-auto
      `}
    >
      {routes.map((route, index) => (
        <NavItem key={`navitem-${index}`} text={route.text} icon={route.icon} />
      ))}
    </nav>
  );
};
export default NavBar;
```

Don’t forget to add your `key` within the `map` function! This is used by React to properly identify each distinct component generated by it.

## NavItem hover states and Tailwind’s group modifier

It would look a little nicer to apply some different styles when a link is hovered. We want to do two things:

* Add a background to the NavLink component being hovered
* Change the icon’s color when its NavLink is hovered

For the first one, we can simply set `hover:bg-gray-600` on the parent in NavItem . But how can we change only the icon element’s color, based on the hover state of the parent element?

We will use Tailwind’s `group` modifier ([more info here](https://tailwindcss.com/docs/hover-focus-and-other-states#styling-based-on-parent-state)), assigning the group class to the top-level element of NavItem and then leveraging `group-hover:text-gray-100` at the icon level.

```jsx
// src/components/NavItem.js
const NavItem = ({ text = "Default", icon = "" }) => {
  return (
    <div className="group flex-grow lg:flex-grow-0 flex flex-col lg:flex-row items-center p-4 cursor-pointer rounded-lg hover:bg-gray-600">
      <div className="flex justify-center items-center text-4xl text-gray-400 group-hover:text-gray-100">
        {icon}
      </div>
      <p className={`text-sm lg:text-lg lg:ml-4 text-gray-200`}>{text}</p>
    </div>
  );
};

export default NavItem;
```

## Add a header to NavBar for large screens

Create file `src/components/NavHeader.js`

```jsx
// src/components/NavHeader.js
import { HiChevronDoubleLeft } from "react-icons/hi";

const NavHeader = () => {
  return (
    <div className="flex items-center justify-between p-4">
      <div className="flex items-center mr-12">
        <div className="flex items-center justify-center w-12 h-12 rounded-lg bg-gray-600 text-2xl">
          J
        </div>
        <div className="flex flex-col p-2">
          <h6 className="text-lg text-gray-200">The Junto</h6>
          <p className="text-sm text-gray-300">Tailwind Demo</p>
        </div>
      </div>
      <div className="text-gray-400 text-3xl cursor-pointer hover:text-gray-300">
        <HiChevronDoubleLeft />
      </div>
    </div>
  );
};

export default NavHeader;
```

Here we’re adding a space where we could put a logo, some stacked text where we can have our company name or something, as well as some subtext.

On the right side of the header we’ll use the `HiChevonDoubleLeft` icon from `react-icons` to act as a collapse button, minimizing the sidebar into a thinner column.

Jump back to NavBar and let’s import and render our new shiny header

```jsx
// src/components/NavBar.js
import routes from "../constants/routes";
import NavItem from "./NavItem";
import NavHeader from "./NavHeader";
const NavBar = () => {
  return (
    <nav
      className={`...`}
    >
      <NavHeader />
      {routes.map((route, index) => (...))}
    </nav>
  );
};

export default NavBar;
```

Looks great! On a large screen at least. We need to fix the responsiveness a bit here.

Back in NavHeader let’s hide the NavHeader completely on small screens using Tailwind media queries

```jsx
// src/components/NavHeader.js
// ...
const NavHeader = () => {
  return (
    <div className='hidden lg:flex items-center justify-between p-4'>
		// ...
		</div>
// ...
```

There’s still one other problem, which we can see if we resize our browser width to around the `1000px` mark. Our header is getting smushed! Not too long after the smush-point our `sm:` queries kick in, but there’s a few hundred pixels where it looks bad, and the chevron can’t be seen.

Where there’s a flex, there’s a way! We’re going to jump back into NavBar and set one more property here `lg:flex-shrink-0` .

```jsx
// src/components/NavBar.js
const NavBar = () => {
  return (
    <nav
      className={`
			bg-gray-900 text-gray-100

      lg:flex-shrink-0

      flex flex-row justify-around
			lg:flex-col lg:justify-start
      w-full lg:w-[18rem]

      lg:p-2
      lg:overflow-y-auto
      `}
    >
      <NavHeader />
      {routes.map((route, index) => (...))}
    </nav>
  );
};

export default NavBar;
```

This property sets `flex-shrink: 0;` on our NavBar component, which is actually an “upwards-facing” property, it is telling the resultant `nav` element not to shrink within it’s parent element’s flex behavior

We can look back in our Layout to see what that means. Our Layout, on large screens, is operating as a `flex-row` with two elements: the `nav` output of NavBar and the `div` we’re wrapping around its `{children}` . So as the screen is shrunk, we’re telling this flex container not to shrink our `nav` and so our `div` will shrink instead

```jsx
// src/components/Layout.js
import NavBar from "./NavBar";
const Layout = ({ children }) => {
  return (
    <div
      className={`
      h-screen w-screen overflow-hidden
      flex flex-col-reverse lg:flex-row
      lg:overflow-y-auto lg:overflow-x-hidden
      `}
    >
      <NavBar />
      <div className="flex flex-col overflow-y-auto">{children}</div>
    </div>
  );
};
export default Layout;
```

## Set up sidebar “toggling” between a minimized version and the full version

In this step we’re aiming for our NavBar to have a UX with the following needs:

* On large screens, where we’re rendering a sidebar, we want to be able to expand and shrink the NavBar
* When expanded, it’ll show the full NavItem icon and text, and have a minimize button
* When shrunk, we’ll only show the NavItem icon, and have an expand button
* We don’t want our small screen NavBar to change at all

To manage our sidebar state (shrunk or expanded), let’s leverage React’s `useState` hook. In NavBar lets get that set up, and we’ll set it’s initial state to false so that we start out expanded.

```jsx
// src/components/NavBar.js
import { useState } from "react";
// ...
const NavBar = () => {
  const [isMinimized, setIsMinimized] = useState(false);
// ...
```

If we think about what’s going to be changing, we can pick out a couple main areas that will need to be aware of this hook:

* Our header, so that we can control it’s minimized state, and have our icon button toggle the state
* Each NavItem, so that we can determine whether to render the text or not

We’ll pass those in appropriately:

```jsx
// src/components/NavBar.js
import { useState } from "react";
import routes from "../constants/routes";
import NavItem from "./NavItem";
import NavHeader from "./NavHeader";

const NavBar = () => {
  const [isMinimized, setIsMinimized] = useState(false);
  return (
    <nav
      className={`
			bg-gray-900 text-gray-100

      lg:flex-shrink-0

      flex flex-row justify-around
			lg:flex-col lg:justify-start
      w-full lg:w-[18rem]

      lg:p-2
      lg:overflow-y-auto
      `}
    >
      <NavHeader
        onToggle={() => setIsMinimized((current) => !current)}
        isMinimized={isMinimized}
      />
      {routes.map((route, index) => (
        <NavItem
          key={`navitem-${index}`}
          text={route.text}
          icon={route.icon}
          isMinimized={isMinimized}
        />
      ))}
    </nav>
  );
};

export default NavBar;
```

We’re just going to establish a `onToggle` function for the NavHeader to hit that changes state to the opposite of what it currently is.

Let’s hook our collapse button up first so that we can see it in action. In NavHeader we’ll accept our new properties, and put them to work. We’ll give the `div` around our `HiChevronDoubleLeft` icon an `onClick` handler, and just directly call the `onToggle()` function.

```jsx
// src/components/NavHeader.js
import { HiChevronDoubleLeft } from "react-icons/hi";

const NavHeader = ({ onToggle, isMinimized }) => {
  return (
    <div className="hidden lg:flex items-center justify-between p-4">
      {!isMinimized && (
        <div className="flex items-center mr-12">
          <div className="flex items-center justify-center w-12 h-12 rounded-lg bg-gray-600 text-2xl">
            J
          </div>
          <div className="flex flex-col p-2">
            <h6 className="text-lg text-gray-200">The Junto</h6>
            <p className="text-sm text-gray-300">Tailwind Demo</p>
          </div>
        </div>
      )}
      <div
        className="text-gray-400 text-3xl cursor-pointer hover:text-gray-300"
        onClick={() => onToggle()}
      >
        <HiChevronDoubleLeft />
      </div>
    </div>
  );
};

export default NavHeader;
```

Now when we click minimize, the rest of our NavHeader content disappears!

While we’re here, let’s have our icon change so that it’s facing the right way when in the minimized state. We could use a different icon and do a ternary operator like `{ issMinimized ? (...) : (...) }`, but it’ll be more fun to use the same icon, but have it rotate. We can animate the rotation, too.

```jsx
// src/components/NavHeader.js
import { HiChevronDoubleLeft } from "react-icons/hi";

const NavHeader = ({ onToggle, isMinimized }) => {
  return (
    <div className="hidden lg:flex items-center justify-between p-4">
      {!isMinimized && (
        <div className="flex items-center mr-12">
          <div className="flex items-center justify-center w-12 h-12 rounded-lg bg-gray-600 text-2xl">
            J
          </div>
          <div className="flex flex-col p-2">
            <h6 className="text-lg text-gray-200">The Junto</h6>
            <p className="text-sm text-gray-300">Tailwind Demo</p>
          </div>
        </div>
      )}
      <div
        className={`text-gray-400 text-3xl cursor-pointer hover:text-gray-300 transition-[transform] duration-200 ${
          isMinimized && "rotate-180"
        }`}
        onClick={() => onToggle()}
      >
        <HiChevronDoubleLeft />
      </div>
    </div>
  );
};

export default NavHeader;
```

On the `div` around our icon, we’ve updated the `className` to use what’s called a [template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) using backticks. To pass that to `className` we need to wrap it in curly braces. This let’s us easily leverage Javascript within the string! We can do that with the `${ ... }` syntax.

In this case, we’re using `${ isMinimized && 'rotate-180'}` to add the `rotate-180` class to the string if our `isMinimized` state is true.

We’ll use Tailwind’s `transition` property to enable transitions for any of the following properties. We could just set this class, and be done with it, as the default version of `transition` applies to all of the following (by default, and can be modified through a theme extension): `color`, `background-color`, `border-color`, `text-decoration-color`, `fill`, `stroke`, `opacity`, `box-shadow`, `transform`, `filter`, `backdrop-filter`.

Since `rotate-180` translates to plain CSS as `transform: rotate(180deg);`, we actually don’t need our CSS to be prepared to animate anything other than `transform` so we can specify `transition-[transform]` to be explicit about what we need for performance purposes.

Now there’s one more thing I want to do here: We’ll notice that when we toggle our sidebar, the icon is jumping around a bit, vertically. This is because the overall height of our component is changing when we hide all of the other NavHeader content! Gross.

There’s probably a number of ways we could handle this, but let’s KISS (keep it simple, stupid) and just set a permanent height on this component, basically the size of, or slightly larger than, the natural component’s size. `h-20` (aka 5rem) looks good.

```jsx
// src/components/NavHeader.js
// ...
const NavHeader = ({ onToggle, isMinimized }) => {
  return (
    <div className='hidden lg:flex items-center justify-between p-4 lg:h-20'>
			// ...
		</div>
// ...
```

Now let’s hop over to NavItem and do something similar for each NavItem’s text. We could take the same approach as we did previously, and hide the NavItem text if the `isMinimized` state is `true` but that would actually have a side effect!

We are using that text on our small viewport version of NavBar. If you were to minimize the sidebar on a large screen, then shrink the screen, the text would now be gone from the small screen’s NavBar. Also gross, even if it’s kind of an edge case.

Let’s take a different approach here. First we’ll accept our `isMinimized` prop, and then we’ll leverage another template literal to dynamically apply the `lg:hidden` class to our text’s `p` element.

```jsx
// src/components/NavItem.js
const NavItem = ({ text = "Default", icon = "", isMinimized }) => {
  return (
    <div className="group flex-grow lg:flex-grow-0 flex flex-col lg:flex-row items-center p-4 cursor-pointer rounded-lg hover:bg-gray-600">
      <div className="flex justify-center items-center text-4xl text-gray-400 group-hover:text-gray-100">
        {icon}
      </div>
      <p
        className={`text-sm lg:text-lg lg:ml-4 text-gray-200 ${
          isMinimized && "lg:hidden"
        }`}
      >
        {text}
      </p>
    </div>
  );
};
export default NavItem;
```

Excellent - all of the stuff in our NavBar hides itself when minimized, and our small-screen NavBar looks the same.

One problem remains: the sidebar doesn’t actually shrink!

Open up our NavBar again, and update our NavBar width dynamically. We’ll use `auto` when it’s minimized, and set our `w-[18rem]` otherwise

```jsx
// src/components/NavBar.js
// ...

const NavBar = () => {
  const [isMinimized, setIsMinimized] = useState(false);
  return (
    <nav
      className={`
			bg-gray-900 text-gray-100

      lg:flex-shrink-0

      flex flex-row justify-around
			lg:flex-col lg:justify-start
      w-full ${isMinimized ? "lg:w-auto" : "lg:w-[18rem]"}

      lg:p-2
      lg:overflow-y-auto
      `}
    >
      <NavHeader
        onToggle={() => setIsMinimized((current) => !current)}
        isMinimized={isMinimized}
      />
      {routes.map((route, index) => (...))}
    </nav>
  );
};

export default NavBar;
```

## Animate our sidebar when expanding or shrinking

For the piece-de-resistance, let’s animate this bad boy so it feels a little nicer when opening and closing.

We don’t need anything fancy here - Tailwind has everything we need. We’ll again use the `transition` class, although this time we *need to* explicitly state the property we want to target (width), since it isn’t part of the default list of affected properties. So we’ll set `lg:transition-[width]` and we’ll also give it a duration of `lg:duration-200` to extend it a bit, but keep it snappy ([default = 150ms](https://github.com/tailwindlabs/tailwindcss/blob/master/stubs/defaultConfig.stub.js#L864))

```jsx
// src/components/NavBar.js
// ...
const NavBar = () => {
  const [isMinimized, setIsMinimized] = useState(false);
  return (
    <nav
      className={`
			bg-gray-900 text-gray-100

      lg:flex-shrink-0

      flex flex-row justify-around
			lg:flex-col lg:justify-start
      w-full ${isMinimized ? "lg:w-auto" : "lg:w-[18rem]"}

      lg:p-2
      lg:overflow-y-auto
      lg:transition-[width] lg:duration-200
      `}
    >
      <NavHeader ... />
      {routes.map((route, index) => (...))}
    </nav>
  );
};
// ...
```

And voila! Just kidding, it’s not animating still. Why is that?

It is because we’re relying on `auto` for our width setting in our minimized state, and CSS just can’t even with that apparently.

We’ll work around this by setting a defined `width` for our minimized state as well. `h-20` (5rem) feels too small, and `h-24` (6rem) feels too big. We could add an intermediate sizing property into our Tailwind theme for 5.5rem, but let’s just explicitly provide the `rem` value for now via `lg:w-[5.5rem]`

```jsx
// src/components/NavBar.js
// ...

const NavBar = () => {
  const [isMinimized, setIsMinimized] = useState(false);
  return (
    <nav
      className={`
			bg-gray-900 text-gray-100

      lg:flex-shrink-0

      flex flex-row justify-around
			lg:flex-col lg:justify-start
      w-full ${isMinimized ? "lg:w-[5.5rem]" : "lg:w-[18rem]"}

      lg:p-2
      lg:overflow-y-auto
      lg:transition-[width] lg:duration-200
      `}
    >
      <NavHeader ... />
      {routes.map((route, index) => (...))}
    </nav>
  );
};

export default NavBar;
```

Animation is in place, but now we can see our NavHeader gets a little janky during the transition, starting the content all squished up as the NavBar expands.

There’s a few ways we could go about this, such as transitioning the content in and out in various ways, but that feels overkill. For now let’s keep it breezy and use our handy `flex-shrink-0` property again on the outer `div` that we are conditionally rendering in NavHeader

```jsx
// src/components/NavHeader.js
import { HiChevronDoubleLeft } from "react-icons/hi";

const NavHeader = ({ onToggle, isMinimized }) => {
  return (
    <div className="hidden lg:flex items-center justify-between p-4 lg:h-20">
      {!isMinimized && (
        <div className="flex-shrink-0 flex items-center mr-12">
          <div className="flex items-center justify-center w-12 h-12 rounded-lg bg-gray-600 text-2xl">
            J
          </div>
          <div className="flex flex-col p-2">
            <h6 className="text-lg text-gray-200">The Junto</h6>
            <p className="text-sm text-gray-300">Tailwind Demo</p>
          </div>
        </div>
      )}
      <div
        className={`text-gray-400 text-3xl cursor-pointer hover:text-gray-300 transition duration-200 ${
          isMinimized && "rotate-180"
        }`}
        onClick={() => onToggle()}
      >
        <HiChevronDoubleLeft />
      </div>
    </div>
  );
};

export default NavHeader;
```

## Adding a Header into the Layout

We’re looking good, but I want to add one more thing: a header that will show above our content. This is usually necessary for things like profile buttons, other primary actions and/or navigation purposes.

Lets create our Header file, and we’ll give it a `w-full` so it spans the page nicely, and just add some text and a horizontal line to act as a separator.

```jsx
// src/components/Header.js
const Header = () => {
  return (
    <div className="w-full p-4">
      <h1 className="text-lg mb-4">The Junto</h1>
      <hr />
    </div>
  );
};
export default Header;
```

Pop into Layout, and let’s import and render it. But where should we be placing it in our JSX?

We want it to sit above our content on both large and small screens, so we’ll place it inside of the `div` element, right above `{children}` for now.

```jsx
// src/components/Layout.js
import NavBar from "./NavBar";
import Header from "./Header";
const Layout = ({ children }) => {
  return (
    <div
      className={`
      h-screen w-screen overflow-hidden
      flex flex-col-reverse lg:flex-row
      lg:overflow-y-auto lg:overflow-x-hidden
      `}
    >
      <NavBar />
      <div className="flex flex-col overflow-y-auto">
        <Header />
        {children}
      </div>
    </div>
  );
};
export default Layout;
```

# Concluding thoughts

We’ve made it! Let’s review what we’ve done:

* We’ve got a slick layout that is responsive and can serve as a great base for lots of different types of web applications.
* We’ve used smart abstraction of components and introduced loops to keep our Tailwind from becoming messy and duplicative.
* Our sidebar expand / minimize functionality is smoothly animated, and can easily be extended to house more complexity.
* We’ve used `rem` units everywhere, instead of `px` so that our app keeps its proportions well even when zoomed in.
* Our main content and our sidebar can both scroll vertically if needed, independent of each other.

# Resources

* [Tailwind Docs](https://tailwindcss.com/docs/installation)
* [Create-React-App Docs](https://create-react-app.dev/docs/getting-started)
* [React-Icons](https://react-icons.github.io/react-icons/)
* [Guide: EM vs REM vs PX](https://engageinteractive.co.uk/blog/em-vs-rem-vs-px)

