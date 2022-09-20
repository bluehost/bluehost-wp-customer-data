# bluehost-wp-customer-data
Bluehost Specific Customer Data Module Integrations.

This package integrates with the module [endurance-wp-data-module](https://github.com/bluehost/endurance-wp-module-data/) and integrates bluehost specific customer data into the module to better serve customers. 

This is not a full module but a package that hooks into an existing module in the module system. See the [module loader](https://github.com/bluehost/endurance-wp-module-loader/) for more details about modules.

## Data flow

The guapi endpoint doesn't return properly unless the site's domian has fully resolved (which takes some time on intitial install) and the site is hosted on bluehost proper (not other brands or even bh india). The bluehost plugin expects the site installer to provide certain customer data (id, plan, signupdate etc) as json data in the `bh_cdata_guapi` option when the site is initially installed (since version 1.4). Historically this data was then saved as a transient for a week (and deleted) before checking the API endpoint for updates. However, we found that deleting the provided data was not optimal and opted to soft expire the data and save as an option as to not lose any valid data.

The data is now (since version 1.5) converted to a persisting `bh_cdata` option as a serialized object (rather than a transient). There is an additional option `bh_cdata_expiration` for a soft expiration in 30 days time (MONTH_IN_SECONDS) at which point we attempt to fetch fresh data from the guapi api and saved to the `bh_cdata` option.

The module checks for customer data in the following order:
- customer data in option `bh_cdata`.
- customer data in legacy transient `bh_cdata`.
- as provided data (from installer).
- Finally if not found anywhere else, (or if the option is expired) data is fetched from the guapi endpoint which is then saved to the option for future use (with soft expiration).

Note: The requests to the guapi endpoint are throttled when they don't return valid date. This helps us control how much various sites (with the plugin installed) ping the api. In the case where a site is not on bluehost it will send 10 requests per week attempting to collect customer information. When data is stored, the soft expiration is set to a full month, and will not delete the stale data until valid data is returned.
