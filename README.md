# BACKSTAGE LAB

# Index

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Customising Backstage](#customising-backstage)
  - [Plugins I've Added](#plugins-ive-added)
  - [Templates I've Added](#templates-ive-created)
- [Initial Setup](#initial-setup)
- [Local Backstage Setup](#backstage-local-setup)
- [Run In Kubernetes Environment](#run-in-kubernetes-environment)
- [Excersise](#run-in-kubernetes-environment)
  - [What we are starting off with](#what-we-are-starting-off-with)
  - [What we are doing](#what-we-are-doing)


</br>
</br>

# Introduction
This is a spinoff of my [Automate All The Things](https://github.com/tferrari92/automate-all-the-things) project. While working on the [Nirvana Edition](https://github.com/tferrari92/automate-all-the-things-nirvana) which will include an IDP built with Backstage, I'm creating this smaller lab for anyone who wants to start experimenting with this tool.

We'll be using a GitOps methodology with Helm, ArgoCD and the App Of Apps Pattern. There is some extra information [here](/docs/argocd-notes.md), but you are expected to know about these things.

</br>
</br>

# Prerequisites
- Minikube installed
- kubectl installed
- Helm installed

</br>
</br>

# Customising Backstage
Backstage is made to be customizable. You are supposed to modify it in ways that fit your own needs. I've already added some custom stuff to the default Backstage installation that I think are essential. 

## Plugins I've added
### [Kubernetes plugin](https://backstage.io/docs/features/kubernetes/)
Kubernetes in Backstage is a tool that's designed around the needs of service owners, not cluster admins. Now developers can easily check the health of their services no matter how or where those services are deployed — whether it's on a local host for testing or in production on dozens of clusters around the world.

It will elevate the visibility of errors where identified, and provide drill down about the deployments, pods, and other objects for a service.

### [GitHub Discovery plugin](https://backstage.io/docs/integrations/github/discovery) 
The GitHub integration has a discovery provider for discovering catalog entities within a GitHub organization. The provider will crawl the GitHub organization and register entities matching the configured path. This can be useful as an alternative to static locations or manually adding things to the catalog. This is the preferred method for ingesting entities into the catalog.

I've installed it without events support. Updates to the catalog will rely on periodic scanning rather than real-time updates.

</br>

## Templates I've created
### New nodejs in new repo
lorem ipsum

### New nodejs in existing repo
lorem ipsum

### New backstage user
lorem ipsum

### New backstage group
lorem ipsum

### New documentation

</br>
</br>

# Initial Setup
In order to turn this whole deployment into your own thing, we need to do some initial setup:

1. Fork this repo. Keep the repository name "backstage-minikube-lab".
1. Clone the repo from your fork:

```bash
git clone https://github.com/<your-github-username>/backstage-minikube-lab.git
```

2. Move into the directory:

```bash
cd backstage-minikube-lab
```

2. Run the initial setup script. Come back when you are done:

```bash
python3 python/initial-setup.py
```

4. Hope you enjoyed the welcome script! Now push your customized repo to GitHub:

```bash
git add .
git commit -m "customized repo"
git push
```

</br>
</br>

# Backstage Local Setup
Before deploying Backstage in a Kubernetes environment (Minikube), we need to build it locally. Testing the change you make to your Backstage implementation is also recommended to be done locally since it's much quicker than building the image, pushing it, etc. to test in in K8S.

### Install NVM
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
nvm install 18
nvm use 18
nvm alias default 18
```

### Install yarn
```bash
npm install --global yarn
yarn set version 1.22.19
yarn --version
yarn global add concurrently
```

### Get GitHub PAT (Personal Access Token)
lorem pisum

### Local testing
Create en env var for your GitHub token
```bash
export GITHUB_TOKEN=<your-github-token>
```

Then run
```bash
cd backstage/my-backstage/
yarn install
yarn tsc
yarn dev
```

This should open backstage in your browser. If all runs smoothly, we can proceed to deploying backstage in Minikube. 

Every time you make changes to the Backstage code, it's recommended you test it by running it locally with "yarn dev", since it will be much faster that testing it in Minikube.

"Ctrl + C" to stop the running process and let's continue...

</br>
</br>

# Run in Kubernetes Environment

### Build and push backstage container image to DockerHub
To build and push the Docker image, run the build-push-image.sh script
```bash
chmod +x build-push-image.sh
./build-push-image.sh
```

### Update image tag in backstage chart values
Update the value of backstage.image.tag in the backstage values-custom.yaml
```bash
cd ../..
vim helm/infra/backstage/values-custom.yaml
```

Save and push to repo
```bash
git add .
git commit -m "Updated backstage image tag"
git push
```

### Minikube Environment Setup
Run the start.sh script to get everything setup
```bash
chmod +x backstage/deploy-k8s-environment.sh
backstage/deploy-k8s-environment.sh
```

Now go to localhost:8080 on your browser and Voilá!

</br>
</br>


# Excersise

## What we are starting off with
We are starting off with a Redis database and a backend. Everytime the backend recieves a request it gets the value of "count" from the Redis db and returns it to the user. Before returning it, it adds +1 to "count".

You can test it like this:
```bash
kubectl get pods -n my-app-dev -l app=my-app-backend-dev -o name | xargs -I {} kubectl exec -n my-app-dev {} -- curl -s localhost:3000
```

You should get:
```bash
{"count":1}%
```

You can test it on the other environments too:
```bash
kubectl get pods -n my-app-stage -l app=my-app-backend-stage -o name | xargs -I {} kubectl exec -n my-app-stage {} -- curl -s localhost:3000
kubectl get pods -n my-app-prod -l app=my-app-backend-prod -o name | xargs -I {} kubectl exec -n my-app-prod {} -- curl -s localhost:3000
```

## What we are doing
We are going to create the missing piece with the help of backstage, the frontend.

Let's analyze the backend. With this Gitops setup we have, there's a number of things that need to exist in order for the backend service to be deployed. These are:
1. The [my-app/backend directory](/my-app/backend/): In a production environment, the backend service would have its own repo where we would store all the application code. In this small lab we'll just save it in its own directory.
2. The [helm/my-app/backend directory](/helm/my-app/backend/): Here we save the Helm chart for our backend service. This of course would also be in its own repo on a production environment.
3. The [backend service argocd application manifests](/argo-cd/applications/my-app/backend/): These are read by the App of Apps to 
4. The [backend build pipeline]():

All of these files and directories we need to create for any new service we want to deploy. Luckily, we have Backstage Software Templates.












### Backstage needs github api token to access software catalog. Get one on github console.

When creating a personal access token on GitHub, you must select scopes to define the level of access for the token. The scopes required vary depending on your use of the integration: https://backstage.io/docs/integrations/github/locations/#token-scopes
Reading software components:
    repo        
Reading organization data:
    read:org
    read:user
    user:email
Publishing software templates:
    repo
    workflow (if templates include GitHub workflows)



























#### Info interesante:
https://backstage.spotify.com/learn/backstage-for-all/software-catalog/4-modeling/
https://backstage.spotify.com/learn/standing-up-backstage/putting-backstage-into-action/8-integration/
https://backstage.spotify.com/learn/onboarding-software-to-backstage/onboarding-software-to-backstage/5-register-component/

#### Info datallada sobre objetos de tipo template:
https://backstage.io/docs/features/software-catalog/descriptor-format#kind-template
#### Aqui las acciones q puede hacer el template:
http://localhost:3000/create/actions
#### Para acciones q no existen default:
https://backstage.io/docs/features/software-templates/writing-custom-actions/
#### A note on RepoUrlPicker
In the template.yaml file of the template we created, you must have noticed ui:field: RepoUrlPicker in the spec.parameters field. This is known as Scaffolder Field Extensions.

These field extensions are used in taking certain types of input from users like GitHub repository URL, teams registered in catalog for the owners field, etc. Such field extensions can also be customized for your own organization. See https://backstage.io/docs/features/software-templates/writing-custom-field-extensions/

#### Aca hay ejemplos de templates:
https://github.com/backstage/software-templates

#### Software Templates at Spotify
At Spotify, we have dozens of Software Templates. We divide them into several disciples like Backend, Frontend, Data pipelines, etc. Inside Spotify, we also have stakeholder groups for Web, Backend, Data, etc. separately. These Software Templates are hosted on our internal GitHub enterprise, maintained and reviewed by the concerned experts in the discipline.

The Technical Architecture Group (TAG) at Spotify is the body responsible for reducing fragmentation by deciding on the various Backend, Frontend, Data frameworks to be used inside Spotify. Hence, new Software Templates with completely new frameworks are carefully discussed and reviewed.

Our Software Templates are fundamental to the concept of Golden Paths at Spotify. The Golden Path is the opinionated and supported way to build something (for example, build a backend service, put up a website, create a data pipeline). The Golden Path Tutorial is a step-by-step instructions that walks you through this opinionated and supported path.

The blessed tools — those on the Golden Path — are visualized in the Explore section of Backstage. Read more https://engineering.atspotify.com/2020/08/how-we-use-golden-paths-to-solve-fragmentation-in-our-software-ecosystem/


#### Component objects:
https://backstage.io/docs/features/software-catalog/descriptor-format/#overall-shape-of-an-entity





# Following this approach, no template will load in the UI if any one of those is broken!!!
  locations:
    - type: url
      target: https://github.com/tomasferrarisenda/backstage/blob/main/templates/all-templates.yaml
      rules:
        - allow: [Template]


## USAR PLUGIN DE AUTODISCOVERY POR LA MODALIDAD ACTUAL NO BORRA USERSAL SACARLOS DEL REPO
# EXPLICAR LO DE app-config.yaml y app-config.production.ymal
uno se usa para local (yarn dev), el otro apra el cluster. Ppalmente por la config para la bbdd postgress (en local no la utilizamos)




# BACKSTAGE
If the only change you've made is to the app-config.yaml (or other configuration files) and not to the application code itself, you don't necessarily need to run yarn build or yarn build:backend. The Docker image build process should copy the updated configuration files into the image.





# COSAS MODIFICADAS:
values custom de argo, ingress.enabled cambiado a false


AGREGARLE DESCRIPTION AL REPO DE GHUB