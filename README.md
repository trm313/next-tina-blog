This is a [Tina CMS](https://tina.io/) project.

## Problems I saw during project setup

I used the Tina Barebones starter, with regular Javascript instead of Typescript for now

You'll need to create an `.env` file, and there is an included `.env.example` file for you

Initially I was struggling to get changes to deploy properly - my site in development and one deployed to Netlify were not the same. For some reason my files weren't being staged for git at first, had to manually add them.

The more confusing aspect was that some of the `.tina/` folder's files weren't being staged either, and there's some rules in here https://tina.io/docs/tina-folder/overview/ although it's missing some info. I had to manually add the `.tina/config.js`, `.tina/__generated__/_graphql.json`, and `.tina/__generated__/_schema.json` files

## Local Development

Install the project's dependencies:

```
yarn install
```

Run the project locally:

```
yarn dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

### Building the Starter Locally (Using the hosted content API)

Replace the `.env.example`, with `.env`

```
NEXT_PUBLIC_TINA_CLIENT_ID=<get this from the project you create at app.tina.io>
TINA_TOKEN=<get this from the project you create at app.tina.io>
NEXT_PUBLIC_TINA_BRANCH=<Specify the branch with Tina configured>
```

Build the project:

```bash
yarn build
```

## Learn More

To learn more about Tina, take a look at the following resources:

- [Tina Docs](https://tina.io/docs)
- [Getting starter guide](https://tina.io/guides/tina-cloud/starter/overview/)

You can check out [Tina Github repository](https://github.com/tinacms/tinacms) - your feedback and contributions are welcome!

## [Deploy on Vercel](https://tina.io/guides/tina-cloud/add-tinacms-to-existing-site/deployment/)
