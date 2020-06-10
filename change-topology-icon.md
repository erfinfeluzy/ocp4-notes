# Change Openshift Topology Icon

You can change topology icon on developer's perspective menu on ocp dashboard.

## Step 1: See static assets on ocp

Open this url on web browser: $OCP_WEB_CONSOLE/static/assets/

There will be list of *.svg assets (eg: camel.svg)

## Step 2: Add label on your application

app.kubernetes.io/name=camel
