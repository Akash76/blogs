# User authorization in Nextjs

Hello everyone, This is Akash! Things are getting better everyday and hope everyone are good. I am working as a MERN full stack developer in TCS. In recent times I came across Nextjs and is one of the best frameworks available for developing web applications. As the official [website](https://nextjs.org/) says, it is React framework
for production. Also it gives all the features you need for production like hybrid static & server rendering, TypeScript support, smart bundling, route pre-fetching etc. In this blog I would like to show you how to use AWS Amplify with Next and how to authorize users.

## Creating new Next app

Creating a new next app is as simple as executing a command in terminal. Make sure you have node installed in your pc.

```sh
$ npx create-next-app nextapp
```

Here "nextapp" is app name. Now open this new app in your favourite code editor.

## Start Next server

Once you open the nextapp directory in any editor, you can observe various files and directories. The directory for our interest is "pages". It is here we create and write code for our web pages. "_app.js" is where our entire website starts at. Simply execute below command to start server.

```sh
$ npm run dev
```

It starts server on port 3000. Open any browser and go to http://localhost:3000 to access your app.

The content of the page is present in pages/index.js file. "pages" is where we create pages for our app and each page is associated with a route based on its file name. For example, content of pages/index.js can be seen at http://localhost:3000/ and pages/about.js at http://localhost:3000/about etc. To know more about pages, refer [this](https://nextjs.org/docs/basic-features/pages)

## Initialising Amplify

AWS Amplify is a set of tools and services that can be used together or on their own, to help front-end web and mobile developers build scalable full stack applications, powered by AWS. You can install amplify cli using npm.

```sh
$ npm install -g @aws-amplify/cli
```

Now initialise amplify at root of our app and stick on to the default configurations as below.

```sh
$ amplify init

? Enter a name for the project nextapp
The following configuration will be applied:

Project information
| Name: nextapp
| Environment: dev
| Default editor: Visual Studio Code
| App type: javascript
| Javascript framework: react
| Source Directory Path: src
| Distribution Directory Path: build
| Build Command: npm run-script build
| Start Command: npm run-script start

? Initialize the project with the above configuration? Yes
? Select the authentication method you want to use: AWS profile
? Please choose the profile you want to use default
```

Next add authentication service

```sh
$ amplify add auth

Do you want to use the default authentication and security configuration? Default configuration
How do you want users to be able to sign in? Username
Do you want to configure advanced settings? No, I am done.
```

Now deploy the service

```sh
$ amplify push --y
```

## Creating pages

Install below packages as we use them in our app in next steps.

```sh
$ npm install aws-amplify @aws-amplify/ui-react @emotion/css
```

In our app, we are gonna maintain a state to store authentication status of a user. Inorder to access that state information in our pages and components, we will make use of useContext react hook. Create a directory libs and file contextLib.js in libs. Write below code in contextLib.js

```js
// libs/contextLib.js
import { useContext, createContext } from "react";

export const AppContext = createContext(null);

export function useAppContext() {
  return useContext(AppContext);
}
```
Here we are creating and exporting a context object called AppContext, a function which will return useContext object. These two objects will be imported in subsequent files in pages. Lets update pages/_app.js.

Replace content of _app.js with below

```js
import {useState, useEffect} from 'react'
import Amplify, { Auth } from 'aws-amplify'
import config from '../src/aws-exports'
import Link from 'next/link'
import {css} from '@emotion/css'
import { AppContext } from "../libs/contextLib";
import '../styles/globals.css'

Amplify.configure({
  ...config,
  ssr: true
})

export default function MyApp({ Component, pageProps }) {
  const [isAuthenticated, setIsAuthenticated] = useState(false)

  useEffect(() => {
    Auth.currentAuthenticatedUser()
      .then(() => {
        setIsAuthenticated(true)
      })
      .catch(err => setIsAuthenticated(false))
  }, [])

  console.log('Auth: ', isAuthenticated)
  return (
    <div>
      <nav className={navStyle}>
        <Link href="/">
          <span className={linkStyle}>About</span>
        </Link>
        <Link href="/home">
          <span className={linkStyle}>Home</span>
        </Link>
        {
          !isAuthenticated &&
          <Link href="/login">
            <span className={linkStyle}>Login</span>
          </Link>
        }
      </nav>
      <AppContext.Provider value={{
        isAuthenticated,
        setIsAuthenticated
      }}>
        <Component {...pageProps} />
      </AppContext.Provider>
    </div>
  )
}

const linkStyle = css`
  margin-right: 20px;
  cursor: pointer;
`

const navStyle = css`
  float: right;
  margin-top: 10px;
`
```
There are few things to notice here. AppContext is imported from libs/contextLib.js. We are creating "isAuthenticated" state and "setIsAuthenticated" function using useState react hook. Also we are making use of Amplify from 'aws-amplify' to configure our app and enabling Amplify SSR.

Component is wrapped with AppContext. isAuthenticated, setIsAuthenticated are passed to the value of AppContext Provider. With this we can access those fields in our components and pages.

Next lets create two components, AuthenticatedRoute and UnauthenticatedRoute, inside components directory at root of app.

```js
// components/AuthenticatedRoute.js
import { useAppContext } from "../libs/contextLib"
import { useRouter } from "next/router";

const AuthenticatedRoute = (WrappedComponent) => {
    return (props) => {
        const { isAuthenticated } = useAppContext();
        const Router = useRouter();
        if (typeof window !== "undefined") {
            if (!isAuthenticated) {
                Router.push("/login");
                return null;
            }
            return <WrappedComponent {...props} />;
        }
        // If we are on server, return null
        return null;
    };
};

export default AuthenticatedRoute;

```

```js
// components/UnauthenticatedRoute.js
import { useAppContext } from "../libs/contextLib"
import { useRouter } from 'next/router'

const UnauthenticatedRoute = (WrappedComponent) => {
    return (props) => {
        const { isAuthenticated } = useAppContext();
        const Router = useRouter();
        if (typeof window !== "undefined") {
            if (isAuthenticated) {
                Router.replace("/home");
                return null;
            }
            return <WrappedComponent {...props} />;
        }  
        // If we are on server, return null
        return null;
    };
};

export default UnauthenticatedRoute
```

In these components, we have imported useAppContext from libs and we are using isAuthenticated state to determine whether user is authenticated or not. Based on the isAuthenticated value, we are returning the page which user want to access or redirecting them to respective default pages. These two components are exported so that pages can be wrapped around them.

replace pages/index.js with below

```js
// pages/index.js
import Link from 'next/link'
import AuthenticatedRoute from '../components/AuthenticatedRoute'
import styles from '../styles/Home.module.css'

function Index() {
  return (
    <div className={styles.container}>
      <h3>
        Index
      </h3>
    </div>
  )
}

export default AuthenticatedRoute(Index)
```
Home.module.css is not modified.

create home.js and login.js in pages.

```js
// pages/home.js
import AuthenticatedRoute from "../components/AuthenticatedRoute";
import { AmplifySignOut } from '@aws-amplify/ui-react'
import { useAppContext } from "../libs/contextLib";

function Home(props) {
    const { setIsAuthenticated } = useAppContext();
    return (
        <div>
            <h1>Homeee!!!</h1>
            <AmplifySignOut style={{
                width: 400
            }} onClick={() => {
                setIsAuthenticated(false)
            }}/>
        </div>
    )
}

export default AuthenticatedRoute(Home);
```

```js
// pages/login.js
import { useState, useEffect } from 'react'
import { Auth } from 'aws-amplify'
import { withAuthenticator } from '@aws-amplify/ui-react'
import { useAppContext } from "../libs/contextLib";
import UnauthenticatedRoute from '../components/UnauthenticatedRoute';

function Login() {
  const { data, setIsAuthenticated } = useAppContext();
  const [user, setUser] = useState(null)
  useEffect(() => {
    // Access the user session on the client
    Auth.currentAuthenticatedUser()
      .then(user => {
        console.log("User: ", user)
        setUser(user)
        setIsAuthenticated(true)
      })
      .catch(err => setUser(null))
  }, [])
  return (
    <div>
      { user && <h1>Welcome, {user.username}: {data}</h1> }
    </div>
  )
}

export default withAuthenticator(UnauthenticatedRoute(Login))
```

It can be observed that home.js is exporting Home component wrapped around AuthenticatedRoute. login.js is exporting Login component wrapped around UnauthenticatedRoute. Also we are using withAuthenticator from '@aws-amplify/ui-react' so that we can signup and signin users. useEffect in login.js is checking if any user is authenticated and based on that it is changing the isAuthenticated value using setIsAuthenticated.

Now restart the server to see all the changes that are made.

## Conclusion

Complete source code for this blog is available [here](https://github.com/Akash76/nextapp). In this blog we saw how to authorize users in Nextjs. Next is getting more attention these days. Getting equiped with new skill is always an advantage.

Stay safe!!!