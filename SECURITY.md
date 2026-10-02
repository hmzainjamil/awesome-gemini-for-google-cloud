# Security and cloud data handling

Cloud tools can access projects, data stores, model endpoints, and billing resources. Before using any linked integration:

- Verify source and current Google Cloud documentation.
- Use a separate project and least-privilege IAM role.
- Protect service account credentials and rotate exposed keys.
- Review where prompts, files, logs, and outputs are sent or stored.
- Confirm billing, quotas, region, and approval behavior before production use.

This list does not audit or endorse linked tools. Report vulnerabilities privately through GitHub if enabled or contact the affected project's maintainer. Do not publish secrets or exploit details.
