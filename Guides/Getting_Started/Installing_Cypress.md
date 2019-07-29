# インストール

https://docs.cypress.io/guides/getting-started/installing-cypress.html#System-requirements

## システム要件

- macOS 10.9以上（64ビットのみ）
- Linux Ubuntu 12.04以上、Fedora 21以上、Debian 8以上（64ビットのみ）
- Windows 7以上

## `npm`でインストール

```
cd ~
mkdir cypress_test
cd cypress_test

npm init
```

- すべてのプロンプトで未入力（デフォルト）のままEnter

```
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (cypress_test)
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
About to write to /home/vagrant/cypress_test/package.json:

{
  "name": "cypress_test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}


Is this OK? (yes)
```

```
npm install cypress --save-dev
```

- 以下のような出力になった。

```

> cypress@3.3.1 postinstall /home/vagrant/cypress_test/node_modules/cypress
> node index.js --exec install

Installing Cypress (version: 3.3.1)

 ✔  Downloaded Cypress
 ✔  Unzipped Cypress
 ✔  Finished Installation /home/vagrant/.cache/Cypress/3.3.1

You can now open Cypress by running: node_modules/.bin/cypress open

https://on.cypress.io/installing-cypress

npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN cypress_test@1.0.0 No description
npm WARN cypress_test@1.0.0 No repository field.

+ cypress@3.3.1
added 189 packages from 124 contributors and audited 329 packages in 129.796s
found 0 vulnerabilities
```
