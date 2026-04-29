## Known Issue: API Permission Locking with Dual UI Permission Silos

### Overview

The ManageIQ API does not properly lock down endpoints when using custom buttons or role-based access controls due to dual permission silos. The API grants access if **either** the Service UI permission **or** the standard UI permission is enabled, which can lead to unintended API access even when one UI permission is disabled.

ManageIQ maintains two separate permission systems for many actions:

1. **Standard UI permissions** - Controls access in the main ManageIQ interface
2. **Service UI (SUI) permissions** - Controls access in the Self-Service UI

The API evaluates both permission sets independently. If **either** permission is granted, the API request succeeds, regardless of which UI the user is accessing.

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

This dual-permission architecture creates a security gap:

- Disabling only the standard UI permission (`service_retire_now`) does **not** prevent API access if the Service UI permission (`sui_services_retire`) remains enabled
- Disabling only the Service UI permission does **not** prevent API access if the standard UI permission remains enabled
- Users may inadvertently grant API access by enabling permissions in only one location

### Workaround

To properly lock down API endpoints and prevent unauthorized access:

#### Step 1: Identify All Permission Identifiers

Review the [`api.yml`](https://github.com/ManageIQ/manageiq-api/blob/a079f12860157f4c754274b9334d660777fbb312/config/api.yml) configuration to identify all permission identifiers associated with the feature you want to restrict.

#### Step 2: Disable Permissions in Both Locations

Navigate to **Access Control** → **Roles** and modify the target role to disable **all** feature identifiers for the action:

1. Locate the standard UI permission (e.g., `service_retire_now` under **Services** → **My Services** → **Operate** → **Retire Services**)
2. Locate the Service UI permission (e.g., `sui_services_retire` under **Service UI** → **My Services** → **Operate** → **Retire Service**)
3. Disable **both** checkboxes to fully restrict the feature

#### Step 3: Verify API Access

Test API requests to confirm that access is properly denied after disabling both permissions.

### Visual Reference

The following screenshot illustrates the dual permission structure in the Access Control interface:

![Dual Permission Checkboxes](../images/retire_service.png)

Note the two separate "Retire Service" permissions:
- One under the standard **Services** section
- One under the **Service UI** section

Both must be disabled to fully lock down the API endpoint.

### Affected Features

This issue affects multiple features with dual permission identifiers. Common examples include:

- Service retirement
- Service creation
- Service modification
- Custom button actions
- Other operations with both standard and Service UI permissions

### Best Practices

When configuring role-based access controls:

1. **Always check both permission locations** - Review both standard UI and Service UI sections
2. **Consult api.yml** - Verify all permission identifiers in the API configuration
3. **Test API access** - Confirm that API requests are properly restricted after permission changes
4. **Document custom roles** - Maintain clear documentation of which permissions are enabled/disabled
5. **Regular audits** - Periodically review role configurations to ensure permissions remain properly locked down
