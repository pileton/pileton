module.exports = {
  name: "howmuch",
  description: "Counts how many messages a user has sent across all accessible channels.",
  commands: [
    {
      name: "howmuch",
      description: "Count a user's messages in all channels you can access.",
      options: [
        {
          type: 6, // USER
          name: "user",
          description: "User to count messages for",
          required: true,
        },
      ],
      execute: async (interaction, { getMessages }) => {
        const targetUserId = interaction.data.options.find(opt => opt.name === "user").value;
        const guild = interaction.guild;
        const member = guild.members.get(interaction.user.id);
        const targetMember = guild.members.get(targetUserId);

        if (!targetMember) {
          return interaction.reply({
            content: "Couldn't find that user in this server.",
            ephemeral: true,
          });
        }

        let totalCount = 0;
        const channels = [...guild.channels.values()].filter(
          c =>
            c.type === 0 && // text channels only
            c.permissionsFor(member).has("VIEW_CHANNEL") &&
            c.permissionsFor(member).has("READ_MESSAGE_HISTORY")
        );

        for (const channel of channels) {
          let before;
          let reachedEnd = false;

          for (let i = 0; i < 50; i++) {
            if (reachedEnd) break;
            try {
              const messages = await getMessages(channel.id, {
                limit: 100,
                ...(before ? { before } : {}),
              });

              if (!messages || messages.length === 0) {
                reachedEnd = true;
                break;
              }

              for (const msg of messages) {
                if (msg.author.id === targetUserId) totalCount++;
              }

              before = messages[messages.length - 1].id;
            } catch (err) {
              // Skip inaccessible channel
              break;
            }
          }
        }

        const displayName = targetMember.nick || targetMember.user?.username || "User";

        interaction.reply({
          content: `**${displayName}** has sent **${totalCount}** messages in this server (only scanned channels you can access).`,
          ephemeral: true,
        });
      },
    },
  ],
};
