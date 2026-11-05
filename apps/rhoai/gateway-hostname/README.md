# MaaS gateway public endpoint

Discover each site's MaaS hostname and public TLS certificate through ACM without storing cluster-specific names in Git.

The ConfigurationPolicy sets the MaaS gateway listeners to the configured prefix under the local OpenShift ingress domain. The native consumer-portal controller uses that HTTPS listener hostname for tenant login and its OAuth callback. Keycloak's redirect-URI reconciler uses the same prefix.

The HTTPS listener also needs a certificate valid for its public hostname. A Service serving certificate covers internal `.svc` names only. The policy adds the default ingress certificate selected by `IngressController/default.spec.defaultCertificate.name`, or `router-certs-default` for the generated OpenShift default. Gateway and Secret are both in `openshift-ingress`, so no certificate copy or cross-namespace reference is needed. `musthave` preserves the existing internal certificate; Istio selects the matching public certificate by SNI. Argo ignores only the listener hostname and certificate references resolved by ACM.

Prerequisites: the default ingress certificate must cover the configured hostname and be readable by the gateway. Renewals within the same Secret are handled by the gateway. If an administrator changes the default certificate's Secret name, check for obsolete certificate references before rollout: this gateway implementation supports at most two references per listener.

The native portal also requires its referenced OIDC client Secret in the tenant configuration namespace (`AITenant.status.tenantNamespace`). The ingress policy delivers the hub credential there and in `ai-tenants`; tenant login does not use the dashboard's OpenShift session.

After reconciliation, verify the canonical HTTPS endpoint with certificate verification enabled, the native auth-proxy Deployment, accepted portal HTTPRoutes, and a login redirect to the tenant's configured OIDC issuer. A responding login page alone does not establish API-key creation or inference success.
