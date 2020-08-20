# Getting Started with Terraform on Azure: Tips and Tricks

Infrastructure development is complex, and there can be many hoops to jump through. Making changes to live infrastructure code always involves some risk and can feel like a game of Jenga. While Terraform is relatively new (initial release in  2014), several proven practices are known in the Terraform community that help deal with some hurdles and complexities. Understanding the trial and errors of those who used Terraform early on allows us to learn from them and be more efficient when we are just starting.  This knowledge increases the chance of success in implementing and using Terraform. 

In this guide, we will review some practical tips and tricks to be mindful of when developing with Terraform. Although these are community proven practices, keep in mind that there is more than one way to do something, and it doesn't necessarily mean that's the best and most efficient way for you.

## Modules

In the software development world, we break up reusable segments of our code into parameterized functions and reuse them. This practice allows us to write tests for these functions and maintain them. In Terraform, we use modules in the same manner. We make templates of infrastructure and convert them into modules, which allows the code in each module to be reusable, maintainable, and testable. 

Do not create Terraform configurations that are thousands of lines of code. It reduces code quality and clarity when debugging or making changes. Splitting up your infrastructure code into modules will also prevent you from copying and pasting code between environments, which can introduce many errors. 

###  Version Providers and Modules

The Azure Terraform provider is changing extremely fast. Check out the [change log](https://github.com/terraform-providers/terraform-provider-azurerm/blob/master/CHANGELOG.md) for the Azure provider. The amount of changes made every month is extreme, and many code-breaking changes appear in many updates. To guard yourself against this, version your provider and save yourself the headache:
```
provider "azurerm" {
  version = "1.38.0"
}
```

Additionally, version your modules, especially ones from the Terraform Registry. If you're developing private modules, version those as well. Versioning modules allow for introducing module changes without affecting the infrastructure that is currently using them. For example, let's say my current environment uses version 1.1 of my server module, which is stored in a repo and tagged with v1.1:

```
#Create Server
module "server" {
  source    = "git::https://allanore@dev.azure.com/allanore/terraform-azure-server/_git/terraform-azure-server?ref=v1.1"

  servername    = "myserver123"
  rgname    = azurerm_resource_group.rg.name
  location  = azurerm_resource_group.rg.location
}
```

I need to add a new feature to the module that includes Private Link functionality. In this case, I can use module versioning to safely deploy infrastructure using the new version without affecting infrastructure using version 1.1 by tagging it as version 1.2 and sourcing the specific module version:
 ```
#Create Server
module "server" {
  source    = "git::https://allanore@dev.azure.com/allanore/terraform-azure-server/_git/terraform-azure-server?ref=v1.2"

  servername    = "myserver211"
  rgname    = azurerm_resource_group.rg.name
  location  = azurerm_resource_group.rg.location
  private_link = true
}
```
Using versioning for both providers and modules is a must in Terraform, and you will quickly find out why if your not using them.

### Use Dependency Injections

When reusing modules throughout different environments, some environments may contain required components that already exist. For example, if I write a module that requires a storage account for the service that it's deploying, there may be some environments where this storage account already exists. This scenario may cause some people to attempt to write logic into their code to check if a resource exists or not and perform X action if it does. Introducing complex logic like this is not in line with the declarative methodology that Terraform uses. The resource either exists or not. 

Instead, use dependency injections. Create the module to allow input from resources that either already exist or are created in the configuration.
Take a look at the code below, for example. We have a Network Security Group module that requires a subnet ID to associate the NSG to a subnet. In this example, we are creating the subnet within the same configuration and passing it along. The subnet does not exist prior, so we are creating one to assign to the NSG:
```
# Subnet is directly managed in the Terraform configuration

resource "azurerm_subnet" "snet" {
  name                 = "snet-cloudapp-1"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefix       = "10.0.2.0/24"
}

module "nsg" {
  source                = "./modules/nsg"
  resource_group_name   = azurerm_resource_group.rg.name
  nsg_name              = "nsg-${var.system}-httpsallow"
  source_address_prefix = ["VirtualNetwork"]
  predefined_rules = [
    {
      name     = "HTTPS"
      priority = "500"
    },
    {
      name     = "RDP"
      priority = "501"
    }
  ]

  subnet_id = azurerm_subnet.snet.id
```

Alternatively, we have another environment where a subnet is already existing. We would use the `azurerm_subnet` data source to collect the subnet id information and pass it through to our module using `data.arurerm_subnet.snet.id`: 
```
#Subnet already exists and is called with a data source block

data "azurerm_subnet" "snet" {
  name                 = "snet-coudapp-1"
  virtual_network_name = "production"
  resource_group_name  = "rg-networking"
}

module "nsg" {
  source                = "../../modules/nsg"
  resource_group_name   = azurerm_resource_group.rg.name
  nsg_name              = "nsg-${var.system}-httpsallow"
  source_address_prefix = ["VirtualNetwork"]
  predefined_rules = [
    {
      name     = "HTTPS"
      priority = "500"
    },
    {
      name     = "RDP"
      priority = "501"
    }
  ]

  subnet_id = data.azurerm_subnet.snet.id
}
```
We are not hard coding logic into the module to check for an existing subnet in these two examples. Instead, we take the declarative approach that Terraform is designed for and state in our configuration if it already exists or if it doesn't. Our module can now be reusable in different situations, and we are not complicating the module. We also have better visibility in the module code. Another co-worker on the team can look at the module and get a clear distinction between the two environments.


## Remote State

Try to use [remote state](https://cloudskills.io/blog/terraform-azure-04) as soon as possible in your Terraform development. It can save many headaches later on, especially when multiple people become involved with deploying and managing the same Terraform code. 

Using remote state allows us to secure sensitive variables in our configurations. The Terraform state file is not encrypted, so keeping it on a local workstation may quickly become a security issue. Also, don't make a habit of storing Terraform state files in source control. It increases the chance of exposing sensitive variables, especially if the repository is public. Instead, use a [gitignore](https://git-scm.com/docs/gitignore) file to omit any tf.state files from accidentally getting committed automatically.

### Split Up Terraform States

[Terraservices](https://www.slideshare.net/opencredo/hashidays-london-2017-evolving-your-infrastructure-with-terraform-by-nicki-watt) is a popular term coined a few years ago which involves splitting up Terraform state into different environments to reduce the blast radius on changes made. You don't want to keep all your eggs in one basket. Let's say a team member makes a change to resize a VM. They end up fat fingering the resource group name, and their pipeline workflow auto applies the incorrect change. Terraform rebuilds the resource group and deletes all items causing catastrophic failures to the environment. This situation is not uncommon. 

Also, be aware that your Terraform plan becomes longer and longer if you don't split up a reasonably large environment into separate states. It introduces a new type of risk. Now, the Terraform plan can take longer to run and become harder to read as there are more resources affected by the change. It also means unwanted changes can be easily missed. It's easier to catch a mistake in a few lines of code vs. 10000 lines.

Ideally, you want to separate high-risk components from components that are typically changed and modified. Don't keep all the eggs in one basket. Below is a Terraform project folder structure inspired by [Gruntwork's recommended setup](https://blog.gruntwork.io/how-to-manage-terraform-state-28f5697e68fa):

```
prod
  └ rg
    └ main.tf
    └ variables.tf
    └ output.tf
  └ networking
  └ services
      └ frontend-app
          └ main.tf
          └ variables.tf
          └ output.tf
      └ backend-app
  └ data-storage
      └ sql
      └ redis
```

In the folder structure above, each folder separates out the Terraform states. The resource group has its own state, limiting the risk of daily changes made to the resource group. Services like SQL and Redis are also separated to reduce the risk of accidentally modifying the databases on any change. Splitting up environment states like this reduces a lot of risks. However, it adds a lot of complexity to the infrastructure code. We now have to design ways to feed information between each state and deal with dependencies. Terraform currently doesn't allow for an easy way to manage this. But, tools like [Terragrunt](https://terragrunt.gruntwork.io/), developed by Gruntwork, address handling the complexities with splitting up Terraform state.

## Source Control

Terraform and source control go together hand in hand. If you're not storing your Terraform code in source control, you're missing out on the following benefits:
 - **Change Tracking**: The historical data of all infrastructure changes is extremely powerful and a great bonus for auditors or compliance requirements.
 - **Rollback**: One of the major benefits of infrastructure as code is the ability to "rollback" to the previous configuration or state. However, depending on the environment, that rollback may mean a rebuild. Storing the Terraform configuration in source control makes it easier to re-deploy a pre-existing working state of the environment.
 - **Collaboration Among Teams**: Most source control tools like Azure DevOps, Github, or Bitbucket provide a form of access control. This role-based access allows for separate teams to manage their infrastructure code or provide read-only access to other teams for increased visibility of how the environment works. No more guessing if a firewall port is open or not; look at the code and see if it is.
 - **Automation**: There are many CI/CD tools available that hook into source control. These tools amplify the development and deployment of Terraform.

There is also the concept of *GitOps*, where processes are automated through Git workflows like submitting a pull request. There are community tools out there like [Atlantis](https://www.runatlantis.io/) that are amazing for GitOps with Terraform and can increase efficiency among teams.

### Structure Repo to Business Needs

Designing the source control repo structure for infrastructure can be an intimidating task, especially for those making the jump from a traditional systems engineer to an infrastructure developer role. There are various strategies for storing Terraform code. Some companies put all their Terraform configurations into a single repository, some store configurations with each project's application source code. So which one should I pick?

This short answer is, it depends on your environment. Large environments are going to have a completely different set up than start-up environments. Also, team structure comes largely into play here. Do you have a team that manages all the infrastructure, or is it the developers and DevOps engineers who manage the infrastructure for their application? You will see many DevOps experts and thought leaders in the community talk about [Conway's Law](https://www.thoughtworks.com/insights/blog/applying-conways-law-improve-your-software-development), which states that the communication structure of organizations is the limiter on the way that they develop and design software. This concept is pretty evident when implementing Terraform into your organization. Analyze how your teams are structured and structure your Terraform configuration repos in a way that compliments that structure.  

Here are several common repo strategies:

- **Single Repo:**: All live infrastructure code is in one single repository managed by a governing team.

- **One Repo Per Project:** Every application has its own Terraform folder, and code is stored in a folder of the application source code.

- **One Repo Per Environment:** Environments are split up into their own repository and managed by separate teams. For example, code managing the company firewalls are in a separate repo and managed by the security or networking team. This strategy allows each team to own and manage their infrastructure responsibilities and delegate out lesser permissions for other teams to request changes or view the environment.

Don't stress out over getting your Terraform repo structure right when your first starting out. This will most likely change several times due to business needs, scaling up, or finding a better solution for your environment. Starbucks changed up its repo structure three times over several years and ended up settling [on a repo per component](https://www.hashicorp.com/resources/terraform-at-starbucks-infrastructure-as-code-for-software-engineers/) strategy.

### Keep All Live Infrastructure in the Master Branch

All live infrastructure changes should always stay in the master branch. Storing the same infrastructure code in multiple branches can cause conflicts and create headaches. For example, let's say a team member branches off of master and adjusts the Terraform configuration to change a VM's size. They make their change and deploy it, but don't merge their branch back into master because they are still making changes. A few minutes later, someone else modifies the same VM's tags but creates a different branch off of master that hasn't been updated yet with the new VM size. The change to the tags is deployed, and now the VM size is reverted back to its original size because it didn't contain the VM resize code. This is why it's important to make sure the master branch is always a live representation of the environment. 

### Execute Terraform Code Through a Pipeline

When first starting on Terraform, it is typical to have each infrastructure developer manage the infrastructure by authenticating locally on their machine with the Azure provider (either with AZ Cli or some environment variables). They execute the Terraform code with their local install of Terraform. Long term, this can cause a few headaches like inconsistent Terraform versions among developers. It's best to shift to deploying code with a pipeline by storing Terraform configurations in source control and running a Continuous Integration process that executes the Terraform code on pull requests. A pipeline significantly increases automation capabilities and has a few advantages:

  - Terraform code is run on the same platform every time, reducing errors due to inconsistent dependencies like Terraform versions.
  - Pipelines can introduce configuration error checking and Terraform policy, preventing insecure or destructive configurations changes from being made.
  - Automated testing can run to perform regression tests against modules when a new change is made to the modules.
  - Many pipeline tools provide some sort of secret store functionality that makes it easy to securely pass variables through to Terraform configurations.

## Keeping Designs Simple and Reusable

It's essential to keep the right balance between creating conditional logic and introducing too many complexities. For example, it may be useful to add logic into a networking module that will automatically choose the next available subnet space on a Virtual Network and create a subnet. While this logic prevents a user from having to specify a subnet address when they use the module, it also adds more complexity and can make the module more brittle. It may be better to design the module to contain an argument to take in input for the subnet address, requiring the user to calculate a subnet address for the module input beforehand. These are trade-offs with pros and cons to each.

A *code review* is a software development practice where multiple developers check each other's code for mistakes. With infrastructure development, this is starting to become a more common practice.  Complex Terraform code will take away from the benefits of code reviews.  When peers cannot easily understand the code to review, errors can be easily missed.  

Complex Terraform code will also make it harder to troubleshoot issues and onboard new people to the team. One of the benefits of IaC is the living documentation that it provides. Don't put in logic that makes infrastructure code too complex to use for documentation. 

Connecting inputs and outputs between modules and states can introduce many complexities and can grow to become a dependency nightmare. When passing data between modules or state files, be mindful of the purpose and limit the dependencies involved in your design. 

### Use Provisioners Sparingly

Most provisioners introduce platform or network constraints into our Terraform code. For example, using a provisioner to SSH into a server once it's provisioned and run a script will now require the node executing the Terraform code to have network access to the VM during deployment.

Instead, take advantage of Azure's custom script extension for VMs to [pass a script through to the VM](https://github.com/CloudSkills/Terraform-Projects/blob/master/10-Advanced-HCL/6.%20Dynamic_blocks/main.tf) without any network constraints. 

The goal is to create infrastructure code that you can execute from anywhere. Aim to achieve this as much as possible to give your design even more reusability. 



### Use Terraform Graph to Troubleshoot Dependency Issues

During Terraform development, you may run into resource timing errors where a resource is deployed but relies on another resource that hasn't completed provisioning yet. Maybe a disk or a storage account provisions too fast half of the time or a subnet isn't deployed before a network interface. Typically this is due to a dependency issue in the configuration and is usually solved using interpolation between the proper resources or using a "depends on" block. However, these can be difficult to track down. With `terraform graph`, you can run this command against a configuration directory, and it will produce a DOT format output. You can then copy and paste the output into a website like [WebGraphViz](http://www.webgraphviz.com/) to generate a visual representation of the configuration dependencies to help troubleshoot. 


###  Use the Terraform Registry 

Especially when first starting out, don't try to reinvent the wheel. The [State of the DevOps report](https://cloud.google.com/devops/state-of-devops) shows that highly efficient teams re-use other people's code. There are many Azure modules already created on the [Terraform Registry](https://registry.terraform.io/). If you need to deploy a specific Azure service, take the time to search the registry and see if a module has already been created for the service you need. If the modules that are in the Terraform registry don't meet your needs, you can fork these modules and customize them to your own.

## Conclusion

When getting started with Terraform, don't try to do everything all at once. Start small and try to make minor improvements to your infrastructure little by little. Only focus on making one quality change at a time, instead of building one big massive project from the start with pipelines, modules, tests, and remote state storage.  In the end, you will achieve faster results and create a higher quality design overall.

Also, keep in mind that every environment is different. Not all of these tips will fit every Terraform use case. For example, if your environment is very simple and extremely small, it may not be worth it to split up the Terraform state files. Having good judgment and design for your infrastructure code comes into play. Learn the different concepts in the community and explore how other people are using Terraform, and then do what works best for your environment. 

Infrastructure as code has not yet reached its maturity and has yet to become the standard way of operating for most companies.  Over the years, research has shown that companies adopting infrastructure as code are functioning at significantly higher speeds than those that are still running on traditional methods. This research is making skillsets with tools like Terraform high in demand for companies. Taking the time to learn it is well worth it. Terraform is still in its infancy stage, and the game will continue to evolve and always get better each year. Enjoy the creativity and embrace the complexity and learning that comes with infrastructure development.  
