# disabled

Apps in this directory are intentionally excluded from the cluster.

The parent app-of-apps (`bootstrap/apps.yaml`) only picks up `apps/*.yaml` directly,
so anything moved here will not be deployed.

To re-enable an app, move it back to `apps/`:

    mv apps/disabled/<app>.yaml apps/
