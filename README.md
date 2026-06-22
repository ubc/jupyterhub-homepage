# UBC Jupyter Open — Custom JupyterHub Templates

Custom Jinja2 HTML templates for [Jupyter Open](https://open.jupyter.ubc.ca), UBC's shared JupyterHub environment operated by the [Learning Technology Innovation Centre](https://ltic.ubc.ca/).

Each HTML file overrides the corresponding default in [JupyterHub's template directory](https://github.com/jupyterhub/jupyterhub/tree/master/share/jupyterhub/templates).

## Structure

```
templates/
  login.html       # Landing/login page
  page.html        # Nav bar logo override (all post-login pages)
extra-assets/
  css/
    login.css      # Custom styles
  images/
    ubc-logo.png   # UBC logo
```

## Deployment

Templates are loaded into the JupyterHub pod at startup via a Kubernetes `initContainer` that clones this repo. The cloned content is mounted into the hub container via a shared `emptyDir` volume.

Add the following to your Z2JH `values.yaml`:

```yaml
hub:
  initContainers:
    - name: templates-clone
      image: alpine/git
      args:
        - clone
        - --depth=1
        - --single-branch
        - --
        - https://github.com/ubc/jupyter-homepage
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
    custom-templates: |
      c.JupyterHub.template_paths = ['/usr/local/share/jupyterhub/custom_templates/']
```

## Acknowledgements

This repository is forked from [pangeo-custom-jupyterhub-templates](https://github.com/pangeo-data/pangeo-custom-jupyterhub-templates). The landing page design and structure is inspired by [2i2c's default-hub-homepage](https://github.com/2i2c-org/default-hub-homepage), as adopted by [UC Berkeley](https://github.com/ucbds-infra/datahub-homepage) and the [University of Toronto](https://github.com/2i2c-org/default-hub-homepage).
