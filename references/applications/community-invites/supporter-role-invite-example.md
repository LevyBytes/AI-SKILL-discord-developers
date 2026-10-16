# Supporter role invite example
Context: Discord developer reference: Supporter role invite example
[Source](using-community-invites-index.md) / [Previous source section](prerequisites.md) / [Following source section](what-s-happening.md)
<a id="source-coding-the-invite-1"></a>

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

