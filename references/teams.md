# Teams

> Discord Developers reference. Original prose; identifiers preserved verbatim.

Teams let multiple developers or other Discord users work together on the same app while sharing access to that app's configuration, management controls, and payout settings.

### Creating a Team

 Before you can create a team or join one, your Discord account must have [2FA enabled](https://support.discord.com/hc/en-us/articles/219576828-Setting-up-Two-Factor-Authentication). With 2FA active, go to the [Teams page](https://discord.com/developers/teams) and choose "New Team". (Figure: Screenshot of the Teams page) After creation, Discord opens the **Team Information** page, where you can enter team details and invite other Discord users. Anyone added to a team can reach the apps owned by that team, so invite members carefully. Inviting and removing users is limited to the team Owner and team Admins.

### Adding Apps to a Team

 After the team exists, you can either create apps under that team or move existing apps to it. A team can own up to 75 apps.

### Creating an App

 When creating an app for team ownership, pick the team in the **Team** dropdown in the creation modal. To leave ownership with your individual account, select `Personal`: (Figure: Screenshot of the create application modal with a Team selected)

### Transferring an App

 To move an existing app into a team, open the [Application](https://discord.com/developers/applications) you plan to transfer. On the app's [**General Information** page](https://discord.com/developers/applications/select/information), use the "Transfer App to Team" control near the bottom. A transferred app *cannot* be moved back out of the team. (Figure: Screenshot of where to find the button to transfer an Application to a team)

### Team Member Roles

 A team member has one of four roles: owner, admin, developer, or read-only. Each higher role includes the permissions of the roles below it. Configure member roles from **Team Members** in the team's settings.

###### Team Member Role Types

| Role Name | Value | Description |
| --- | --- | --- |
| Owner* |  | Owners have the broadest permissions and may perform irreversible destructive operations, including deleting team-owned apps or deleting the team. Each team can have only 1 owner. |
| Admin | admin | Admins are granted access close to owners, but destructive operations against the team or its apps are not available to them. |
| Developer | developer | Developers may view team-owned app details, including the client secret and public key, and may perform limited app operations such as setting interaction endpoints or resetting the bot token. A Developer role member *cannot* administer the team, administer members, or perform destructive actions on team-owned apps. |
| Read-only | read_only | Read-only members may view team details and information for team-owned apps, such as application IDs and payout exports. They may also invite private bots that belong to team-owned apps. |

 * The owner role does not appear in the `role` field on the [team member object](https://docs.discord.com/developers/topics/teams#data-models-team-member-object). To determine the team owner, read `owner_user_id` on the [team object](https://docs.discord.com/developers/topics/teams#data-models-team-object).

### Data Models

###### Team Object

| field | type | description |
| --- | --- | --- |
| icon | ?string | Icon image hash for the team |
| id | snowflake | Unique ID of the team |
| members | array of team member objects | Members of the team |
| name | string | Name of the team |
| owner_user_id | snowflake | Current owner's user ID for the team |

###### Team Member Object

| field | type | description |
| --- | --- | --- |
| membership_state | integer | The user's team membership state |
| team_id | snowflake | ID of the team that contains this member |
| user | partial user object | The user's avatar, discriminator, ID, and username |
| role | string | Role of the team member |

###### Membership State Enum

| name | value |
| --- | --- |
| INVITED | 1 |
| ACCEPTED | 2 |
