+++
date = '2025-03-17T13:12:41+01:00'
draft = true
title = 'Copilot'
+++

# GitHub Copilot - My AI Pair Progammer

In this blog post, I will concentrate on the main GitHub Copilot features, its benefits and how to use them. 

Frankly, I am not the best programmer myself, and I am  coming from a platform engineering background rather than being a full-fledged developer. Which makes me a good test candidate for Copilot. 

The promise is GitHub Copilot helps developers code faster and achieve better overall productivity. All you need is a free GitHub account to get started, sign up [here](https://github.com/signup). 

The official documentation can found [here](https://docs.github.com/en/copilot).

GitHub Copilot is available for various IDEs such as:

- Visual Code
- Visual Studio Code
- Xcode
- JetBrains IDEs (e.g. IntelliJ)
- Eclipse (public preview)

There are different subscriptions for GitHub Copilot including a [Free](https://docs.github.com/en/copilot/managing-copilot/managing-copilot-as-an-individual-subscriber/about-github-copilot-free) version with the following limitations:

- Code completions are limited to 2000 completions per month.
- Copilot Chat is limited to 50 chat messages per month

## Features


### Code Completion


### Copilot Chat

TIP: @workspace allows you to ask questions about your entire codebase!

### Copilot Edits


### Copilot in the CLI

To uses this capability we need to install the gh cli and the gh cli copilot extension.

If you are using Mac OS you can install the gh cli via homebrew and the following command ```brew install gh```

Afther the installation you can run the following commands to authenticate and install the copilot extension:
```
gh auth login && gh extension install github/gh-copilot
```

Subsequently you can ask for suggestions for explanations for a specific command, e.g.:

```
➜ gh copilot explain "sudo apt-get" 

Welcome to GitHub Copilot in the CLI!
version 1.1.0 (2025-02-10)

I'm powered by AI, so surprises and mistakes are possible. Make sure to verify any generated code or suggestions, and share feedback so that we can learn and improve. For more information, see https://gh.io/gh-copilot-transparency

Explanation:                                                                                                                                                  
                                                                                                                                                              
  • sudo is used to run commands with elevated privileges, allowing access to system files that require administrative rights.                                
  • apt-get is a command-line tool for handling packages in Debian-based systems, such as Ubuntu.                                                             
  • Common sub-commands include:                                                                                                                              
    • update to refresh the list of available packages and their versions.                                                                                    
    • upgrade to upgrade all installed packages to their latest versions.                                                                                     
    • install <package> to install a specific package.                                                                                                        
    • remove <package> to uninstall a specific package.                                                                                                       
  • You can leverage apt-get --help for more information on available options and usage.       
  ```


### Copilot for pull requests


### Copilot Extensions

A GitHub Copilot Extension is an add-on that provides customized capabilities for GitHub Copilot Chat.

For example:

Querying documentation: A Copilot Extension can allow Copilot Chat to query a third-party documentation service to find information about a specific topic.
AI-assisted coding: A Copilot Extension can use a third-party AI model to provide code suggestions.
Data retrieval: A Copilot Extension can allow Copilot Chat to query a third-party data service to retrieve information about a specific topic.
Action execution: A Copilot Extension can allow Copilot Chat to execute a specific action, such as posting to a message board or updating a tracking item in an external system.

A GitHub Copilot Extension is NOT the GitHub Copilot VS Code Extension found in your IDE, but rather an extra capability to enhance it.

## Summary


## Resources