## API Permission Locking with Dual UI Permission Silos

### Overview

The ManageIQ API grants access if **any** of the allowed permission identifiers are enabled. Since many features have both standard UI and Service UI permission identifiers, disabling only one set does not prevent API access.

### Background

The API authorization checks permissions against identifiers defined in [`api.yml`](https://github.com/ManageIQ/manageiq-api/blob/a079f12860157f4c754274b9334d660777fbb312/config/api.yml). If a user has any of the listed permissions, the API request succeeds. Since both the classic UI and Service UI use the same API, features often have dual permission identifiers to support both interfaces.

### Example: Service Retirement Permissions

Consider the service retirement feature, which has two distinct permission identifiers:

#### API Configuration
From [`config/api.yml`](https://github.com/ManageIQ/manageiq-api/blob/a079f12860157f4c754274b9334d660777fbb312/config/api.yml#L4076-L4079):

```yaml
- :name: retire
  :identifier:
  - service_retire_now
  - sui_services_retire
```

#### Permission Definitions

**Standard UI Permission** - [`miq_product_features.yml`](https://github.com/ManageIQ/manageiq/blob/07bc98843050874f9676a15dcf882a328b1b3a4b/db/fixtures/miq_product_features.yml#L450-L453):
```yaml
- :name: Retire Services
  :description: Retire Services
  :feature_type: control
  :identifier: service_retire_now
```

**Service UI Permission** - [`miq_product_features.yml`](https://github.com/ManageIQ/manageiq/blob/07bc98843050874f9676a15dcf882a328b1b3a4b/db/fixtures/miq_product_features.yml#L7215-L7218):
```yaml
- :name: Retire Service
  :description: Retire Service
  :feature_type: control
  :identifier: sui_services_retire
```

### Impact

Disabling only one permission identifier does not prevent API access. Both identifiers must be disabled to fully restrict the feature through the API.

### Workaround

To fully restrict API access:

1. **Identify all permission identifiers** - Check [`api.yml`](https://github.com/ManageIQ/manageiq-api/blob/a079f12860157f4c754274b9334d660777fbb312/config/api.yml) for the feature's permission identifiers
2. **Disable all identifiers** - Navigate to **Access Control** → **Roles** and disable all associated permissions (e.g., both `service_retire_now` and `sui_services_retire` for service retirement)
3. **Verify** - Test API requests to confirm access is denied

### Visual Reference

![Dual Permission Checkboxes](../images/retire_service.png)

The screenshot shows two separate "Retire Service" permissions—one under **Services** and one under **Service UI**. Both must be disabled to restrict API access.

### Affected Features

This issue affects multiple features with dual permission identifiers. Common examples include:

- Service retirement
- Service creation
- Service modification
- Custom button actions
- Other operations with both standard and Service UI permissions

### Best Practices

1. **Check all permission locations** - Review both standard UI and Service UI sections
2. **Consult api.yml** - Verify all permission identifiers for the feature
3. **Test API access** - Confirm restrictions after permission changes
4. **Document custom roles** - Maintain clear records of permission configurations
5. **Audit regularly** - Periodically review role configurations