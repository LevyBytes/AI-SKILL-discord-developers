# Hosting On Cloudflare Workers

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Discord apps can receive client interaction events as [webhooks](https://docs.discord.com/developers/resources/webhook), including events from [application commands](https://docs.discord.com/developers/interactions/application-commands) and [message components](https://docs.discord.com/developers/components/reference). Discord sends each event as a JSON payload to the configured HTTPS endpoint, called the Interactions Endpoint URL in the app settings. This tutorial builds a JavaScript Discord app based on [`r/aww`](https://www.reddit.com/r/aww). The complete app code is available **[on GitHub](https://github.com/discord/cloudflare-sample-app)**.

### Features and technologies used

- [Discord Interactions API](https://docs.discord.com/developers/interactions/receiving-and-responding), with slash commands specifically
- [Cloudflare Workers](https://workers.cloudflare.com/) as the hosting platform
- [Reddit API](https://www.reddit.com/dev/api/) for returning content to the user

---

### Creating an app on Discord

 First, create the app in the [Discord Developer Dashboard](https://discord.com/developers/applications):
- Visit [https://discord.com/developers/applications](https://discord.com/developers/applications)
- Select `New Application`, then enter a name
- Save the **Public Key** and **Application ID** locally because they are needed later
 (Figure: IDs found in app settings)
- Open the [**Bot** tab](https://discord.com/developers/applications/select/bot) from the left sidebar.
- Copy the bot `token` and store it securely, such as in [1password](https://1password.com/) or [lastpass](https://www.lastpass.com/).
 A bot token is visible only once for security. If it is lost, generate a replacement token.

### Adding bot permissions

 Next, grant the bot the [permissions](https://docs.discord.com/developers/topics/permissions) needed to create and use slash commands and to send channel messages.
- Open the [OAuth2 tab](https://discord.com/developers/applications/select/oauth2/url-generator), choose `URL Generator`, and select the `bot` and `applications.commands` scopes.
- Select `Send Messages` and `Use Slash Commands`, then copy the `Generated URL`.
 (Figure: Configuring bot permissions in app settings)
- Open the generated URL in a browser, complete the OAuth flow, and choose the server where the bot will be developed and tested.

### Creating your Cloudflare Worker

 Cloudflare Workers provide a practical hosting option for Discord apps because they include a free tier, a simple development workflow, and managed infrastructure without virtual machines. One limitation is that a Worker cannot access non-ephemeral CDN media; for example, fetching `https://cdn.discordapp.com/attachments/1234/56789/my_image.png` returns a `403` error. Ephemeral CDN media remains accessible from Cloudflare Workers.
- Open the [Cloudflare Dashboard](https://dash.cloudflare.com/)
- Go to the `Workers` tab and create a service with the same name as the Discord bot
- [install the Wrangler CLI](https://developers.cloudflare.com/workers/cli-wrangler/install-update/) and complete its setup.

### Storing secrets

 The production service needs the values saved earlier. Set those variables with:

```
$ wrangler secret put DISCORD_TOKEN
$ wrangler secret put DISCORD_PUBLIC_KEY
$ wrangler secret put DISCORD_APPLICATION_ID

```

 You also need the Guild ID for the server where the app is installed. Find it in the URL for any channel in that server. In `https://discord.com/channels/123456/789101112`, the Guild ID is the first number, **`123456`**. After identifying the Guild ID, set that variable too:

```
$ wrangler secret put DISCORD_TEST_GUILD_ID

```

### Running locally

 This sample depends on the beta version of `wrangler`, which has better ESM support for Cloudflare Workers. Clone the repository and install dependencies first. The project requires [Node.js](https://nodejs.org/en/) v16 or newer:

```
$ npm install

```

### Project structure

 The cloned app has this project layout:

```
├── .github/workflows/ci.yaml -> GitHub Action configuration
├── src
├── ├── commands.js           -> JSON payloads for commands
├── ├── reddit.js             -> Interactions with the Reddit API
├── ├── register.js           -> Sets up commands with the Discord API
├── ├── server.js             -> Discord app logic and routing
├── test
├── ├── test.js               -> Tests for app
├── wrangler.toml             -> Configuration for Cloudflare Workers
├── package.json
├── README.md
├── renovate.json             -> Configuration for repo automation
├── .eslintrc.json
├── .prettierignore
├── .prettierrc.json
└── .gitignore

```

### Registering commands

 Before testing, register the slash commands used by the app. This sample defines `/awwww` and `/invite`. Their names and descriptions are stored separately in `commands.js`:

```
export const AWW_COMMAND = {
  name: 'awwww',
  description: 'Drop some cuteness on this channel.',
};

export const INVITE_COMMAND = {
  name: 'invite',
  description: 'Get an invite link to add the bot to your server',
};

```

 `register.js` contains the command-registration code. Commands may be [registered globally](https://docs.discord.com/developers/interactions/application-commands#create-global-application-command) for every server where the app is installed, or [registered on a single server](https://docs.discord.com/developers/interactions/application-commands#create-guild-application-command). This example concentrates on global commands:

```
import { AWW_COMMAND, INVITE_COMMAND } from './commands.js';
import fetch from 'node-fetch';

/**
 * This file is meant to be run from the command line, and is not used by the
 * application server.  It's allowed to use node.js primitives, and only needs
 * to be run once.
 */

const token = process.env.DISCORD_TOKEN;
const applicationId = process.env.DISCORD_APPLICATION_ID;

if (!token) {
  throw new Error('The DISCORD_TOKEN environment variable is required.');
}
if (!applicationId) {
  throw new Error(
    'The DISCORD_APPLICATION_ID environment variable is required.'
  );
}

/**
 * Register all commands globally.  This can take o(minutes), so wait until
 * you're sure these are the commands you want.
 */
async function registerGlobalCommands() {
  const url = `https://discord.com/api/v10/applications/${applicationId}/commands`;
  await registerCommands(url);
}

async function registerCommands(url) {
  const response = await fetch(url, {
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bot ${token}`,
    },
    method: 'PUT',
    body: JSON.stringify([AWW_COMMAND, INVITE_COMMAND]),
  });

  if (response.ok) {
    console.log('Registered all commands');
  } else {
    console.error('Error registering commands');
    const text = await response.text();
    console.error(text);
  }
  return response;
}

await registerGlobalCommands();

```

### Running the server

 Run this command locally one time before beginning:

```
$ DISCORD_TOKEN=**** DISCORD_APPLICATION_ID=**** node src/register.js

```

 Now run the code locally by starting the development server:

```
$ npm run dev

```

### Setting up ngrok

 A slash command causes Discord to make an HTTP request to a public endpoint. Local development needs a public tunnel, so this tutorial uses [`ngrok`](https://ngrok.com/) to expose the local server.

```
$ npm run ngrok

```

 (Figure: ngrok forwarding address) `ngrok` forwards traffic from a public endpoint to the local machine. Copy the HTTPS URL it provides; it will look similar to `https://8098-24-22-245-250.ngrok.io`. Return to the Discord Developer Dashboard and set the app's `Interactions Endpoint URL` to that value. (Figure: Interactions Endpoint URL) Use this setup for local testing. After deploying to Cloudflare, **update this field to the Cloudflare Worker URL.**

### Deployment

 The repository is configured to deploy automatically to Cloudflare Workers when changes reach the `main` branch. For manual deployment, run `npm run publish`; it invokes `wrangler publish`. A GitHub Action deployment needs an [API Token and your Account ID from Cloudflare](https://developers.cloudflare.com/workers/cli-wrangler/authentication/). Store those values [as secrets in the GitHub repository](https://docs.github.com/en/actions/security-guides/encrypted-secrets) so GitHub Actions can use them. The `.github/workflows/ci.yaml` example below shows the configuration:

```
release:
  if: github.ref == 'refs/heads/main'
  runs-on: ubuntu-latest
  needs: [test, lint]
  steps:
    - uses: actions/checkout@v2
    - uses: actions/setup-node@v2
      with:
        node-version: 16
    - run: npm install
    - run: npm run publish
      env:
        CF_API_TOKEN: ${{ secrets.CF_API_TOKEN }}
        CF_ACCOUNT_ID: ${{ secrets.CF_ACCOUNT_ID }}

```

### Code deep dive

 The main app logic is in `src/server.js`. Cloudflare Workers use an exported `fetch` function as the entry point for each request. This server code validates that incoming requests are legitimate Discord requests, then passes them to a router so handling logic is easier to manage.

```
export default {
  /**
   * Every request to a worker will start in the `fetch` method.
   * Verify the signature with the request, and dispatch to the router.
   * @param {*} request A Fetch Request object
   * @param {*} env A map of key/value pairs with env vars and secrets from the cloudflare env.
   * @returns
   */
  async fetch(request, env) {
    if (request.method === 'POST') {
      // Using the incoming headers, verify this request actually came from discord.
      const signature = request.headers.get('x-signature-ed25519');
      const timestamp = request.headers.get('x-signature-timestamp');
      const body = await request.clone().arrayBuffer();
      const isValidRequest = verifyKey(
        body,
        signature,
        timestamp,
        env.DISCORD_PUBLIC_KEY
      );
      if (!isValidRequest) {
        console.error('Invalid Request');
        return new Response('Bad request signature.', { status: 401 });
      }
    }

    // Dispatch the request to the appropriate route
    return router.handle(request, env);
  },
};

```

 In this sample, every Discord API call is POSTed to `/`. The app uses the [`discord-interactions`](https://github.com/discord/discord-interactions-js) npm module to parse incoming events and send responses.

```
/**
 * Main route for all requests sent from Discord.  All incoming messages will
 * include a JSON payload described here:
 * /developers/docs/interactions/receiving-and-responding#interaction-object
 */
router.post('/', async (request, env) => {
  const message = await request.json();
  console.log(message);
  if (message.type === InteractionType.PING) {
    // The `PING` message is used during the initial webhook handshake, and is
    // required to configure the webhook in the developer portal.
    console.log('Handling Ping request');
    return new JsonResponse({
      type: InteractionResponseType.PONG,
    });
  }

  if (message.type === InteractionType.APPLICATION_COMMAND) {
    // Most user commands will come as `APPLICATION_COMMAND`.
    switch (message.data.name.toLowerCase()) {
      case AWW_COMMAND.name.toLowerCase(): {
        console.log('handling cute request');
        const cuteUrl = await getCuteUrl();
        return new JsonResponse({
          type: InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
          data: {
            content: cuteUrl,
          },
        });
      }
      case INVITE_COMMAND.name.toLowerCase(): {
        const applicationId = env.DISCORD_APPLICATION_ID;
        const INVITE_URL = `https://discord.com/oauth2/authorize?client_id=${applicationId}&scope=applications.commands`;
        return new JsonResponse({
          type: InteractionResponseType.CHANNEL_MESSAGE_WITH_SOURCE,
          data: {
            content: INVITE_URL,
            flags: InteractionResponseFlags.EPHEMERAL,
          },
        });
      }
      default:
        console.error('Unknown Command');
        return new JsonResponse({ error: 'Unknown Type' }, { status: 400 });
    }
  }

  console.error('Unknown Type');
  return new JsonResponse({ error: 'Unknown Type' }, { status: 400 });
});

```

### Next steps

 The full sample remains available [on GitHub](https://github.com/discord/cloudflare-sample-app). After the app is built and deployed, use these ideas to customize it:
- Add more interactive behavior with **[message components](https://docs.discord.com/developers/components/reference)** such as buttons and select menus.
- Explore additional **[public APIs](https://github.com/public-apis/public-apis)** on GitHub.
- Join the **[Discord Developers server](https://discord.gg/discord-developers)** for API questions, Discord API team events, and developer community discussion.
