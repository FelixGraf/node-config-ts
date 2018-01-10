node-config-ts 
=======
[![Build Status](https://travis-ci.org/tusharmath/node-config-ts.svg?branch=master)](https://travis-ci.org/tusharmath/node-config-ts)
[![Greenkeeper badge](https://badges.greenkeeper.io/tusharmath/node-config-ts.svg)](https://greenkeeper.io/)

A simple configuration manager for typescript based projects.

## Usage

1. Install package 
    ```bash
    npm i node-config-ts
    ```

2. Add a `postinstall` step
    ```json
    {
       "scripts" : {
         "postinstall": "node-config-ts"
       }
    }
    ```

3. Create a `config` directory inside your project's root folder and add a `default.json` file. A typical folder structure looks as follows —  
    ```
    root/
    └── config/
        └── default.json
    ```
    `default.json` should contain your application's configuration

4. Create typings
    ```bash
    npm install
    ```
    A new `Config.d.ts` will be generated automatically. This file could be ignored from git as it gets automatically generated based on the structure of `default.json`

5. Import and use `node-config-ts`
    
    ```typescript
    import {config} from 'node-config-ts'
   
    console.log(config) // logs the config data from default.json    
    ```

# Configuration
The configs are merged in the following order of priority —

1. Commandline params
2. Environment variable
2. User specific config file
3. Deployment specific config file
4. Environment specific config file

## Configuring using files
Configurations are loaded via config files that are written in JSON format for now. A typical project looks like this —

```
root/
└── config/
    ├── Config.d.ts
    ├── default.json
    ├── deployment/
    │   ├── staging.example.com.json
    │   ├── production.example.com.json
    │   └── qa.example.com.json
    ├── env/
    │   └── production.json
    └── user/
        ├── ec2-user.json
        ├── andy.json
        └── sara.json
```

There are three directories in which a project can have configurations — `deployment`, `env` and `user`. These directories can have multiple files inside them and based on the environment variables an appropriate config file is selected for overriding the base `default.json`. For example if the `NODE_ENV` variable is set to `production` the `env/production.json` configuration will be merged with `default.json` and override default values with its own. Similarly if `DEPLOYMENT` env variable is set to `staging.example.com` then `deployment/staging.example.com.json`  is merged with the other configs. Here is a table for environment to directory mapping —


| **process.env** | **directory**      |
|-----------------|--------------------|
| NODE_ENV        | /config/env        |
| DEPLOYMENT      | /config/deployment |
| USER            | /config/user       |

## Using environment Variables
Whenever the value is prefixed with the letters `@@` **node-config-ts** automatically looks for an environment variable with that name. For example — 

```json
// default.json
{
  "port": "@@APP_PORT"
}
```
In the above case automatically the value of `port` is set to the value that's available inside the environment variable `PORT`.

```bash
export APP_PORT=3000
node server.js // server started with config.port as 3000
```

## Using commandline params

The command line arguments can override all the configuration params. This is useful when you want to start a node server by passing the port externally —

```bash
node server.js --port 3000
```

In the above case even if the `default.json` has a port setting of `9000` the cli argument can override it
```json
// default.json
{
    "port": 9000
}
```

## Differences with node-config
1. **No reserved words:** With [node-config] you can not use a certain set of [reserved words] in your configuration. This is an unnecessary restriction and `node-config-ts` doesn't have it.
2. **Simpler API:** Instead of using methods such as `config.get('xxx')` in `node-config` you can simply use the exported `config` object.
3. **Warnings & Errors:** [node-config] relies on calling the `get` and the `has` methods to issue errors. This is unsafe typically when the configurations are different between your dev and production environments.
With `node-config-ts` you can trust the typescript compiler to issue an error immediately when you try to access a property that isn't defined anywhere. Consider the following case —

    #### default.json
    ```json
    {
      "port": 3000
    }
    ```
    #### user/john.json
    ```json
    {
      "baseURL": "/api"
    }
    ```
    
    In the above case the final configuration *should* look something like this on `john`'s local machine —
    
    ```json
    {
      "port": 3000,
      "baseURL": "/api"
    }
    ```

    ##### reading using node-config:
    ```ts
    import config from 'config'

    console.log(config.get('port'))
    console.log(config.get('baseURL')) // works locally but fails in production
    ```
    This would work when `john` is running the application on his local machine. But as soon as its deployed in production the configuration property `baseURL` isn't available anymore and it results in runtime exceptions.

    ##### using node-config-ts:
    ```ts
    import {config} from 'node-config-ts'

    console.log(config.port) // proper intellisense support
    console.log(config.baseURL) // throws compile time error immediately on local machine.
    ```
    Because the above object `config`, is exposed with proper typings, using invalid configurations results in typescript errors. This would happen on both — `john`'s computer and the production server.
    
[node-config]:     https://github.com/lorenwest/node-config
[reserved words]:  https://github.com/lorenwest/node-config/wiki/Reserved-Words
