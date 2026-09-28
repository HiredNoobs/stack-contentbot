# stack-contentbot

Contentbot deployment for k8s.

The chatbot and worker are Deployments with one replica per node labelled `contentbot_core_chatbot` / `contentbot_core_worker` (set in iac-homelab). They're stateless, updates replace each pod in place so there's never a second chatbot in the channel.

They connect to Redis (`redis-tls.redis-core.svc`) and RabbitMQ (`rabbitmq-broker.rabbitmq-core.svc`) as the `contentbot` user, set in `configs/contentbot.conf`.

## Secrets

- `contentbot_secrets.conf` - the app's secrets, from the deployment's `secrets/`.
- The contentbot user's certs for Redis and RabbitMQ, created by `make-user` in `stack-redis` and `stack-rabbitmq` and pulled from Vault as external secrets. `secrets/external.yml` needs:

  ```yaml
  external:
    production:
      redis_core: [ca.crt, contentbot.crt, contentbot.key]
      rabbitmq_core: [ca.crt, contentbot.crt, contentbot.key]
  ```

  `deploy` creates the `external-redis-core` and `external-rabbitmq-core` Secrets from them and stops if a key is missing. They're mounted with the same file names as on swarm (`/etc/contentbot/tls/redis.crt`, `rabbitmq_ca.crt`, ...).
