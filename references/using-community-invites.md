# Using Community Invites

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Community invites add two API-driven capabilities to Discord invites: `role_ids`, which assigns roles when the invite is accepted, and `target_users_file`, which limits acceptance to selected users. Server admins and community managers who want role invites without writing code should use the [Community Invites guide](https://docs.discord.com/developers/communities/guides/community-invites) for the Discord UI flow. This tutorial covers the programmatic approach for bots and apps. After working through it, you will know how to:
- Use the API to create invites that grant roles
- Use the API to create targeted invites for specific user groups

---

### Understanding Community Invites Features

 Community invites build on the normal Discord invite system with two features useful for game integrations:

### Role Assignment

 With the API, `role_ids` can name one or more Discord roles to apply automatically after a player accepts the invite. This removes the need for manual role assignment. **Example use cases:**
- Giving roles to supporters
- Assigning roles tied to in-game characters or achievements
- Assigning roles that unlock permission-gated channels
 After a user accepts the invite and receives roles, those roles stay on the member even if the invite expires or is deleted. Remove them separately with a bot or through Discord's interface.

### Targeted Invites

 Restricting who may accept an invite is available only through the Discord API. The `target_users_file` parameter is a CSV file of Discord user IDs; only listed users can view and use the invite. **Example use cases:**
- Stopping invite sharing for exclusive achievements, such as a unique invite that grants a 100% completion role
- Pairing the restriction with roles that grant mod/admin privileges
- Pairing the restriction with roles that give paying supporters special roles or channel access
 These capabilities still work alongside normal invite settings such as `max_age`, `max_uses`, and related parameters.

---

### Role Granting Invite Example: Auto-Assigning Player Roles

 This example uses the [channel invite API](https://docs.discord.com/developers/resources/channel#create-channel-invite) to grant roles through an invite. It illustrates how a game developer might connect in-game state back to a Discord server.

### Prerequisites

 Before starting, confirm you have:
- A Discord server for testing
- Developer Mode enabled on your account so role, channel, and user IDs can be copied
- A Discord app with a bot token, created in the [Developer Portal](https://discord.com/developers/applications)
- Node.js installed, or another way for your bot or app to send HTTP requests
- Bot permissions for:
  - **Create Instant Invite** (needed to create invites)
  - **Manage Roles** (needed when `role_ids` automatically assigns roles)

### Coding the Invite

 When players enter your Discord server from the game, the invite can automatically give them a **Player** role for game-only channels. The following code creates that kind of invite:

```
// IMPORTANT: Never hardcode tokens or commit them to version control
// Use environment variables or a secure configuration management system
// Replace the placeholder below with your actual bot token, channel ID, and role ID
const BOT_TOKEN = 'REPLACE_WITH_YOUR_BOT_TOKEN_FROM_THE_DEV_PORTAL';
const CHANNEL_ID = '1234567890123456789';
const ROLE_ID = '9876543210987654321';

const response = await fetch(
    `https://discord.com/api/v10/channels/${CHANNEL_ID}/invites`,
    {
        method: 'POST',
        headers: {
        'Authorization': `Bot ${BOT_TOKEN}`,
        'Content-Type': 'application/json'
        },
        body: JSON.stringify({
        role_ids: [ROLE_ID]
        })
    }
);

if (!response.ok) {
const error = await response.json().catch(() => ({
    message: `HTTP ${response.status}`
}));
throw new Error(
    `Failed to create invite: ${error.message}${error.code ? ` (code: ${error.code})` : ''}`
);
}

const invite = await response.json();
console.log(`Created invite: https://discord.gg/${invite.code}`);

```

### What’s Happening

1. **`role_ids` array**: Holds the Discord role IDs applied when someone accepts the invite. The example uses one role, but the array can contain multiple roles.
2. **Response**: Provides an invite object containing a shareable `code`

### Testing It Out

 1Create a **Player** role in your Discord server2Right click the role and choose **"Copy Role ID"**, then paste it into `ROLE_ID` in the script3Right click the invite channel and choose **"Copy Channel ID"**, then paste it into `CHANNEL_ID` in the script4Paste your bot token into `BOT_TOKEN` in the script5Run the code to create an invite code6Share the `discord.gg/...` link with a test account7When the invite is accepted, the **Player** role is granted automatically! (Figure: Example of an invite granting a Player role)

---

### Target Users Example: Creating Targeted Supporter Invites

 This example uses the [channel invite API](https://docs.discord.com/developers/resources/channel#create-channel-invite) to allow only named users to accept an invite. It shows how a game developer or streamer can grant supporter-only roles and channels inside a community.

### Prerequisites

 Before starting, confirm you have:
- A Discord server for testing
- Developer Mode enabled on your account so role, channel, and user IDs can be copied
- A Discord app with a bot token, created in the [Developer Portal](https://discord.com/developers/applications)
- Node.js installed, or another way for your bot or app to send HTTP requests
- Bot permissions for:
  - **Create Instant Invite** (needed to create invites)
  - **Manage Guild** (needed for targeted invites that use `target_users_file`)
  - **Manage Roles** (needed when `role_ids` automatically assigns roles)

### Coding the Invite

 For a game or community subscription service, create exclusive invites that only paying supporters can redeem. The granted role can expose channels hidden from other server members. The following example creates an invite that gives a role to specific users:

```
// IMPORTANT: Never hardcode tokens or commit them to version control
// Use environment variables or a secure configuration management system
// Replace the placeholder below with your actual bot token, channel ID, and role ID
const BOT_TOKEN = "REPLACE_WITH_YOUR_BOT_TOKEN_FROM_THE_DEV_PORTAL";
const SUPPORTER_CHANNEL_ID = '1234567890123456789';
const SUPPORTER_ROLE_ID = '1111222233334444555';

// In production you would fetch this from your subscription database
// Replace these example user IDs with actual Discord user IDs of your test accounts
const activeSupporterIds = [
    '111111111111111111',
    '222222222222222222',
    '333333333333333333',
];

// Create CSV content with user IDs
const csvContent = activeSupporterIds.join('\n');
const csvBlob = new Blob([csvContent], { type: 'text/csv' });

// Use FormData for multipart/form-data request
const formData = new FormData();
formData.append('target_users_file', csvBlob, 'supporters.csv');

formData.append('payload_json', JSON.stringify({
    role_ids: [SUPPORTER_ROLE_ID]
}));

const response = await fetch(
`https://discord.com/api/v10/channels/${SUPPORTER_CHANNEL_ID}/invites`,
{
    method: 'POST',
    headers: {
    'Authorization': `Bot ${BOT_TOKEN}`
    },
    body: formData
}
);

if (!response.ok) {
    const error = await response.json().catch(() => ({
        message: `HTTP ${response.status}`
    }));

    let errorMessage = `Failed to create invite: ${error.message || response.statusText}`;

    if (error.code) {
        errorMessage += ` (code: ${error.code})`;
    }

    if (error.errors) {
        errorMessage += '\nDetailed errors:\n' + JSON.stringify(error.errors, null, 2);
    }

    throw new Error(errorMessage);
}

const invite = await response.json();

console.log(`Share this link with supporters: https://discord.gg/${invite.code}`);

```

### What’s Happening

1. **`target_users_file`**: CSV content containing the Discord user IDs for paying supporters. Only those users can view and accept the invite.
2. **`role_ids` array**: Automatically applies the **Supporter** role, which grants access to exclusive channels
3. **FormData**: Needed for file uploads; place the remaining parameters inside `payload_json`
 Use `Content-Type: application/json` when sending only JSON parameters such as `role_ids`. If the request uploads `target_users_file`, send it as `multipart/form-data`.

### Setting Up the Supporter Role

 For the smoothest setup, give the **Supporter** role these permissions:
1. Create a **Supporter** role in your Discord server
2. Create a private channel (#supporter-polls or #supporter-chat)
3. Right click the channels and choose **"Edit Channel"**, then configure channel permissions so only members with the **Supporter** role can view and send messages there
  - **View Channel** for channels reserved for supporters
  - **Send Messages** and **Add Reactions**

### Testing It Out

 1Create a **Supporter** role in your Discord server2Right click the role and choose **"Copy Role ID"**, then paste it into `SUPPORTER_ROLE_ID` in the script3Right click the invite channel and choose **"Copy Channel ID"**, then paste it into `SUPPORTER_CHANNEL_ID` in the script4Paste your bot token into `BOT_TOKEN` in the script5Right click each allowed user and choose **"Copy User ID"**, then add those IDs to `activeSupporterIds` in the script6Run the code to create an invite code7Share the `discord.gg/...` link with one user in `activeSupporterIds` and one outside that set t8The allowed `activeSupporterIds` users can accept the invite and receive the **Supporter** role! (Figure: Example of an invite granting a Supporter role) (Figure: Example of an invite that can't be accepted)

---

### Resources

 For the full API reference, consult the invite and channel API documentation.

### Invites

Full reference material for invite objects, endpoints, and parameters

### Create Channel Invite

Detailed reference for the endpoint that creates invites
