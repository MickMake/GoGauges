# Provider discovery

## Discovery mechanism

GoGauge discovers providers through wildcard subscriptions to retained `status` and `catalog` topics. The broker supplies existing retained messages when GoGauge subscribes.

## Registry identity

- `provider_id` is the stable public identity.
- `instance_id` identifies the live process publishing messages.
- All messages accepted for one live provider view must agree on both values.

## Conflict

Overlapping messages for one `provider_id` with different live `instance_id` values create a conflict. GoGauge must not merge them or silently choose a winner.

Exact expiry and recovery timing remains a GoGauge Core question.

## Discovery result

Discovery updates:

- provider registry
- catalogue registry
- catalogue cache dirty state

Discovery does not:

- create gauges automatically
- choose gauge types
- position or rearrange dashboards
- rewrite user configuration

New signals become available for configuration and binding; presentation belongs to other GoGauge design areas.
