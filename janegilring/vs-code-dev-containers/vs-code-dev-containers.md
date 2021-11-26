<!--
This is an article template you can use as a starting point when writing DigitalOcean procedural tutorials about intrastructure. Once you've reviewed the template, delete the comments and begin writing your outline or article. You'll find some examples of our custom Markdown at the very bottom of the template.

As you write, refer to our style and formatting guidelines for more detailed explanations:

- [do.co/style](https://do.co/style)

Use our [Markdown previewer](https://www.digitalocean.com/community/markdown) to review your article's formatting.

Readers should be able to follow your tutorial from the beginning to the end on a DigitalOcean Droplet. Before submitting your article to the editorial team, please be sure to create a new Droplet and test your article from start to finish on it exactly as written. Cut and paste commands from the article into your terminal to make sure there aren't typos in the commands. If you find yourself executing a command that isn't in the article, incorporate it into the article to make sure the reader gets the exact same results. We will test your article and send it back to you if we run into technical problems, which significantly slows down the publication process.
-->


# How To Get Started with VS Code Development containers

<!-- Use Title Case for all Titles -->

<!-- Learn about the title, introduction, and Goals sections at https://do.co/style#title-introduction-and-goals -->

<!-- Learn about formatting headers at https://do.co/style#headers -->

<!-- Our articles have a specific structure. Procedural tutorials follow this structure:

* Title
* Introduction (Level 3 heading)
* Prerequisites (Level 2 heading)
* Step 1 — Doing Something (Level 2 heading)
* Step 2 — Doing Something (Level 2 heading)
...
* Step 5 — Doing Something (Level 2 heading)
* Conclusion (Level 2 heading)


Learn more at https://do.co/style/structure -->

### Introduction

When working with code, whether it is software, Infrastructure as Code or other purposes, there are some common challenges.
One of them is the lead time for onboarding new members to a team or a project, due to prerequisites needed for the tasks the new person will be working on.

In this article, we will use the Infrastructure as Code and cloud automation as our main scenario.

The specific tools needed will of course vary from project to project, but some common ones might be Terraform for declarative configurations, PowerShell for scripting and administrative tasks and VS Code as an editor. Along with these example stack of tools comes additional prerequisites, such as VS Code extensions, PowerShell modules, linting tools and so on. Lastly, there are different versions of both the tools mentioned as well as the related prerequisites such as PowerShell modules.

Another aspect is that people use different platforms, such as Linux, macOS and Windows. And even when using the same operating system, there might be differences related to versions and patch levels.

In the end, there is a whole lot of variations which can make it very challenging to have a consistent development environment.

This is not a problem new to the world of software and infrastructure. Container technologies was born with these exact challenges as one of their main tasks to solve. Portability, being able to run the software consistently across different environments such as on-premises or different cloud vendors, is another advantage containers brings to the table.

So how can containers assist with the development environment challenges discussed above?

In this guide, you will learn how to set up VS Code development containers.

When you're finished, you'll be able to leverage dev containers both for your personal projects as well as for team projects and have a consistent development environment.

## Prerequisites

<!-- Prerequisites let you leverage existing tutorials so you don't have to repeat installation or setup steps in your tutorial.  Learn more at https://do.co/style#prerequisites -->

Before you begin this guide you'll need the following:

- A computer running either Linux, macOS or Windows <!-- Also specify the amount of RAM the server needs if relevant. -->
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Visual Studio Code](https://code.visualstudio.com)

## Step 1 — Install the Visual Studio Code Remote - Containers extension

<!-- For more information on steps, see https://do.co/style/#steps -->

The first thing we are going to do after installing the prerequisites is to install the VS Code Remote - Containers extension.

Open VS Code and navigate to the Extensions item in the sidebar. Next, search for "remote containers":

:::image type="content" source="images/step1-1-install-vs-code-extension.png" alt-text="VS Code extension installation":::

Click on "Remote - Containers" and click the Install button which becomes available after clicking on the button.

**Tip: Alternatively, you may install the "Remote Development" extension pack at the bottom of the above screenshot. This contains the "Remote - Containers" extension as well as to other very useful extensions: "Remote SSH" for connecting to remote SSH targets as well as "Remote - WSL" which makes it very convenient to work with the Windows Subsystem for Linux from VS Code.**

## Step 2 — Create a new folder

Create a new, empty folder with the purpose of creating your first dev container. Open that folder in VS Code.

```
cd ~
mkdir myfirstdevcontainer
code .\myfirstdevcontainer
```

:::image type="content" source="images/step2-1-create-folder.png" alt-text="Creating a new folder":::

Select "Yes, I trust the authors" when the new empty folder is being opened in VS Code.

We are now ready to create our first dev container.

## Step 3 — Create a dev container

Open the [Command pallette](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette) in VS Code by using Ctrl+Shift+P (or alternatively, F1 on Windows) and search for Remote Containers

Choose Add Development Container Configuration Files:

:::image type="content" source="images/step3-add-dev-container-configuration-files.png" alt-text="Add development container configuration files":::

A number of templates is available. You may choose the one you want, but for this demo we will be using the Ubuntu template:

:::image type="content" source="images/step3-2-add-dev-container-configuration-files.png" alt-text="Add development container configuration files":::

Next, we will use the focal distro for our testing purposes:

:::image type="content" source="images/step3-3-add-dev-container-configuration-files.png" alt-text="Add development container configuration files":::

Finally, we may choose to install additional tools into our new container image. Here are the ones I chose:

:::image type="content" source="images/step3-4-add-dev-container-configuration-files.png" alt-text="Add development container configuration files":::

Click Ok when finished.
Next, we will be prompted which version of the tools we selected we want to install. In a real world scenario, we would typically pin specific versions for consistency. For this demo, I chose **latest** for all prompts.

At the end, we should now automatically get a subfolder called .devcontainer along with two files: devcontainer.json and Dockerfile:

:::image type="content" source="images/step3-5-add-dev-container-configuration-files.png" alt-text="Add development container configuration files":::

This is folder is typically stored at the root of a source control repository, hence it becomes available for all other people who have cloned the repository.

# Step 4 - Open the newly created dev container

When VS Code detects the two files we created in the previous step at the root of a folder, it will prompt whether we want to "Reopen in Container":

:::image type="content" source="images/step4-1-open-dev-container.png" alt-text="Open development container":::

Should the "Remote - Containers" extension not be installed, it will also prompt to install that.

An alternative approach to reopen the current folder inside the dev container is to once again open the Command pallette and search for Remote Containers. Next, select "Reopen in Container":

:::image type="content" source="images/step4-2-open-dev-container.png" alt-text="Open development container":::

VS Code will now reopen and the following status will be shown:

:::image type="content" source="images/step4-3-open-dev-container.png" alt-text="Open development container":::

Under the hood, the command `docker build` is now being run against the Dockerfile we created previously. This will result in a local container image becoming available on our machine.

The build step might take from a few seconds until a few minutes depending upon the size of the base image we chose, the number of tools to install, the bandwidth of our internet connection as well as the performance of the local machine.

The build step is only needed whenever a change has been made to the configuration files for the development container, so the next time you open the folder inside the dev container, it will typically open in seconds - similar to when opening a regular local folder.

# Step 5 - Using the dev container

When the build process has completed, you should see the folder opened like before. The difference is that it is now opened within the dev container, which is indicated here:

:::image type="content" source="images/step5-1-using-dev-container.png" alt-text="Using development container":::

You can actually click on the green "Dev Container" buttom to bring up a menu with relevant options such as reopening the folder locally and so on:

:::image type="content" source="images/step5-2-using-dev-container.png" alt-text="Using development container":::

Next, let us explore how extensions comes into play when working with dev containers. Navigate to the Extensions item on the sidebar. You should see the following:

:::image type="content" source="images/step5-3-using-dev-container.png" alt-text="Using development container":::

This means we can have separate VS Code extensions per container, and we don`t have to "clutter" or local VS Code instance with numerous extensions (which may also decrease performance/load times).

To install extensions inside the dev container, simply search and install them from the extensions item as usual. In this example, I will go ahead and install the PowerShell extension by clicking Install on the bottom recommandation.

After installing the PowerShell extension, I will go ahead and create the file test.ps1 in the root of the myfirstdevcontainer folder. When opening the file in the editor, the PowerShell extension kicks in and opens the PowerShell Integrated Console automatically:

:::image type="content" source="images/step5-4-using-dev-container.png" alt-text="Using development container":::

Congratulations! You are now ready to start using PowerShell inside your newly created dev container.

## 6 - Gotchas

### Operating system

A major difference if you are Windows user, is that PowerShell is now running inside a Linux container:

:::image type="content" source="images/step5-5-using-dev-container.png" alt-text="Using development container":::

One option could be swapping the base image to for example the mcr.microsoft.com/powershell:lts-7.2.0-windowsserver-ltsc2022 image which can be found on [Docker Hub](https://hub.docker.com/_/microsoft-powershell).
I have not tested this, but it should work and provide you with PowerShell inside a Windows Server 2022 container image.

However, in this modern world of cloud infrastructure, basic Linux administration is becoming a more and more an important skill. Personally, I went through a learning journey in the past couple of years - ending with the **LFCS: Linux Foundation Certified Systems Administrator** certification. You can read more about my journey in [this article](https://www.powershell.no/linux,/powershell,/devops/2021/02/25/learning-linux.html) on my personal blog.

On a relevant note, there is also [Linux Cloud Engineer Bootcamp](http://cloudskills.io/courses/linux) available on CloudSkills.io.

### Containers are stateless

By nature, containers are stateless. This means that all changes performed inside the container is lost when the container stops and starts. The solution to this is mounting persistent data volumes inside the container.

This is something the VS Code extension handles for us automatically when it comes to the project folder we reopened from the local computer:

:::image type="content" source="images/step6-1-under-the-hood.png" alt-text="Data":::

This means that all data inside this folder will persist, but if you place data in other folders - they will be lost.
Also, if you install software inside the container manually - they will also be gone the next time the container image is rebuilt. If you want to install something persistently, the solution is to add the installation command to the Dockerfile and rebuild the container.

In addition to mounting folders from the local machine like the VS Code extension does automatically for the project folder, it is possible to both add additional bind mounts as well as create docker volumes for persistent storage.

You can read more about this in the [Docker documentation](https://docs.docker.com/storage/) as well as the [VS Code documentation](https://code.visualstudio.com/remote/advancedcontainers/add-local-file-mount).

## 7 - Under the hood

Let us have a deeper look at the two configuration files that the VS Code extension created for us in the previous steps.

`devcontainer.json` contains the settings for the VS Code - Remote Containers extension:

#
```
// For format details, see https://aka.ms/devcontainer.json. For config options, see the README at:
// https://github.com/microsoft/vscode-dev-containers/tree/v0.205.2/containers/ubuntu
{
	"name": "Ubuntu",
	"build": {
		"dockerfile": "Dockerfile",
		// Update 'VARIANT' to pick an Ubuntu version: hirsute, focal, bionic
		// Use hirsute or bionic on local arm64/Apple Silicon.
		"args": { "VARIANT": "focal" }
	},

	// Set *default* container specific settings.json values on container create.
	"settings": {},


	// Add the IDs of extensions you want installed when the container is created.
	"extensions": [],

	// Use 'forwardPorts' to make a list of ports inside the container available locally.
	// "forwardPorts": [],

	// Use 'postCreateCommand' to run commands after the container is created.
	// "postCreateCommand": "uname -a",

	// Comment out connect as root instead. More info: https://aka.ms/vscode-remote/containers/non-root.
	"remoteUser": "vscode",
	"features": {
		"kubectl-helm-minikube": "latest",
		"terraform": "latest",
		"github-cli": "latest",
		"azure-cli": "latest",
		"powershell": "latest"
	}
}
```

As we can see, we can customize it further by adding extensions for example. If adding the id of the PowerShell extension (ms-vscode.PowerShell) to the "extensions" property in the json-file - it will get installed automatically when the image is built.

Here is an example for the extensions I typically use in my Infrastructure as Code dev containers:

```
	"extensions": ["ms-vscode.powershell","ms-vscode.azure-account","hashicorp.terraform","takumii.markdowntable","ms-kubernetes-tools.vscode-kubernetes-tools","ms-kubernetes-tools.vscode-aks-tools"],
```

Next is the `Dockerfile`:

```dockerfile
# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.205.2/containers/ubuntu/.devcontainer/base.Dockerfile

# [Choice] Ubuntu version (use hirsuite or bionic on local arm64/Apple Silicon): hirsute, focal, bionic
ARG VARIANT="hirsute"
FROM mcr.microsoft.com/vscode/devcontainers/base:0-${VARIANT}

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>
```

If you have worked with containers before, there is nothing special about this file. It`s  regular Dockerfile which you can define as you want. In our example, we happened to use a predefined base-image - but we could choose to replace this with your own image. For example, you can choose to use the official PowerShell image as a base - which I actually do for a few of my own dev containers:

```dockerfile
FROM mcr.microsoft.com/powershell:7.2.0-ubuntu-focal

RUN apt-get update && apt-get -y upgrade

RUN apt-get update && apt-get install azure-cli

SHELL ["pwsh", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

RUN Install-Module PSDepend -Force

COPY ["powershell/requirements.psd1", "/tmp/requirements.psd1"]

RUN Invoke-PSDepend /requirements.psd1 -Force

RUN az aks install-cli --install-location /usr/local/bin/kubectl --kubelogin-install-location /usr/local/bin/kubelogin --client-version 1.22.0

# Switch to non-root user:
RUN useradd --create-home vscode
WORKDIR /home/vscode
USER vscode

RUN Install-Module -Name oh-my-posh -Force
RUN Install-Module -Name PSReadLine -Force -AllowPreRelease
```

As you can see, I also install the PSDepend PowerShell module in order to install a set of pre-defined PowerShell modules as defined in requirements.psd1.

Alternatively, you may install modules the regular way using Install-Module as seen on the last two lines in the Dockerfile.

## 8 - Tips and tricks


### Personalizing with dotfile repositories

As stated in the [documentation](https://code.visualstudio.com/docs/remote/containers#_personalizing-with-dotfile-repositories):

>Dotfiles are files whose filename begins with a dot (.) and typically contain configuration information for various applications. Since development containers can cover a wide range of application types, it can be useful to store these files somewhere so that you can easily copy them into a container once it is up and running.

>A common way to do this is to store these dotfiles in a GitHub repository and then use a utility to clone and apply them. The Remote - Containers extension has built-in support for using these with your own containers. If you are new to the idea, take a look at the different dotfiles bootstrap repositories that exist.

### Sharing Git credentials with your container

Also from the [documentation](https://code.visualstudio.com/docs/remote/containers#_sharing-git-credentials-with-your-container):

>The Remote - Containers extension provides out of box support for using local Git credentials from inside a container.

In the documentation, they will walk through the two supported options.

I would encourage you to read through the [documentation](https://code.visualstudio.com/docs/remote/containers#_getting-started) for the *VS Code - Remote containers* extension for additional information and configuration options you might want to leverage.

## Conclusion

In this article you learned how to get started with VS Code Development containers.

The extension makes it very convenient to perform development work inside a running container based on a standardized setup which can be shared across teams using regular source control.

:::image type="content" source="images/step8-1-architecture-containers.png" alt-text="Data":::

*Image credit: Microsoft*

Now you can start using them both for your personal projects as well as team projects in order to benefit from consistent development environments.