# UBC Jupyter Open — Custom JupyterHub Templates

Custom Jinja2 HTML templates for [Jupyter Open](https://open.jupyter.ubc.ca), UBC's shared JupyterHub environment operated by the [Learning Technology Innovation Centre](https://ltic.ubc.ca/).

Each HTML file overrides the corresponding default in [JupyterHub's template directory](https://github.com/jupyterhub/jupyterhub/tree/master/share/jupyterhub/templates).

## Structure

```
templates/
  login.html              # Landing/login page
  page.html               # Nav bar logo override (all post-login pages)
extra-assets/
  css/
    login.css             # Styles for the landing/login page
    page.css              # Styles for post-login pages (nav bar logo light/dark mode)
  images/
    ubc-white-logo.png    # White UBC full signature logo — used on the landing page blue header
    ubc-nav-blue-logo.png # Dark blue UBC logo — used in nav bar in light mode
    ubc-nav-white-logo.png # White UBC logo — used in nav bar in dark mode
```

## Deployment

Templates are loaded into the JupyterHub pod at startup via a Kubernetes `initContainer` that clones this repo. The cloned content is mounted into the hub container via a shared `emptyDir` volume.

Add the following to your Z2JH `values.yaml`:

```yaml
hub:
  initContainers:
    - name: templates-clone
      image: 032401129069.dkr.ecr.ca-central-1.amazonaws.com/docker-hub/alpine/git:latest
      args:
        - clone
        - --depth=1
        - --single-branch
        - --
        - https://github.com/ubc/jupyterhub-homepage
        - /srv/repo
      securityContext:
        runAsUser: 1000
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
      volumeMounts:
        - name: custom-templates
          mountPath: /srv/repo

  extraVolumes:
    - name: custom-templates
      emptyDir: {}

  extraVolumeMounts:
    - mountPath: /usr/local/share/jupyterhub/custom_templates
      name: custom-templates
      subPath: templates
    - mountPath: /usr/local/share/jupyterhub/static/extra-assets
      name: custom-templates
      subPath: extra-assets

  extraConfig:
    01-custom-templates: |
      c.JupyterHub.template_paths = ['/usr/local/share/jupyterhub/custom_templates/']
```

> **Note:** `alpine/git` is pulled via the ECR Docker Hub pull-through cache (`docker-hub/*`). Ensure the image is available in your ECR before deploying.

## Maintenance Banner
The login and post-login pages support an optional full-width maintenance banner. It is controlled entirely within this repo — no Helm changes required, and restarting the hub pod does **not** affect running student pods.

### Activating the banner
1. Open `templates/page.html` and find the `{% block announcement %}` block.
2. Remove the `{# ... #}` Jinja2 comment tags wrapping the banner `<div>` to activate it.
3. Update the message text as needed.
4. Commit and push.
5. Restart the hub pod to pull the latest templates:
```bash
   kubectl rollout restart deployment/hub -n default
   kubectl rollout status deployment/hub -n default
```

The banner will appear at the top of all pages (login, home, spawn, etc.).

### Deactivating the banner
1. Wrap the banner `<div>` back in `{# ... #}` comment tags in `templates/page.html`.
2. Commit and push.
3. Restart the hub pod:
```bash
   kubectl rollout restart deployment/hub -n <namespace>
   kubectl rollout status deployment/hub -n <namespace>
```

### Banner styling
Banner styles are defined in `extra-assets/css/page.css` under the `#maintenance-banner` selector. Both light and dark mode are supported.

## Acknowledgements
This repository is forked from [pangeo-custom-jupyterhub-templates](https://github.com/pangeo-data/pangeo-custom-jupyterhub-templates). The landing page design and structure is inspired by [2i2c's default-hub-homepage](https://github.com/2i2c-org/default-hub-homepage), as adopted by [UC Berkeley](https://github.com/ucbds-infra/datahub-homepage) and the [University of Toronto](https://github.com/2i2c-org/default-hub-homepage).
