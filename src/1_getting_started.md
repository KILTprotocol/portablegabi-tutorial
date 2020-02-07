# Getting Started

In order to get started you need to install [node](https://nodejs.org/en/), [yarn](https://yarnpkg.com/getting-started), [go](https://golang.org/doc/install), [go dep](https://github.com/golang/dep) and [rust](https://rustup.rs).
Make sure you setup your `GOPATH` and `GOBIN` folders and environment variables.
After everything is setup clone the [portablegabi project](https://github.com/KILTprotocol/portablegabi) into the correct go path.

```bash
mkdir -p $GOPATH/src/github.com/KILTprotocol/
git clone https://github.com/KILTprotocol/portablegabi.git \
  $GOPATH/src/github.com/KILTprotocol/portablegabi
cd $GOPATH/src/github.com/KILTprotocol/portablegabi
```

Install all the dependencies, build the portablegabi WASM and compile the typescript code.

```bash
yarn install
yarn build
```

Now you can execute the example to ensure everything worked fine.

```bash
ts-node docs/example.ts
```

## Add portablegabi as dependency

To use portablegabi in your project use either npm or yarn to add it to your dependencies:

- `npm install @kiltprotocol/portablegabi`
- `yarn add @kiltprotocol/portablegabi`
