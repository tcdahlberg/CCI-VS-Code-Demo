# Best practices for agentic coding

While every developer will land on their own technique for working with agentic coding tools, here’s one developers take on running an efficient agentic ship.

Some things about me as a developer:

- I am a Salesforce developer
- I like to use scratch orgs for development and testing
- I like to use CumulusCI for project management and deployments
- I like to use GitHub Copilot for AI-assisted coding
- I like to use IntelliJ IDEA with the [Illuminated Cloud plugin](vibe-coding.md#illuminated-cloud) for my IDE

## Model Context Protocols (MCP) 

MCPs are a way to structure the context you provide to your AI coding assistants, and can include things like code formatting rules, testing guidelines, terminal command best practices, and more. The goal is to create a consistent and comprehensive set of instructions that your AI can refer to when generating code or providing feedback.
Install MCP that make sense for your project.
  - [Salesforce MCP](vibe-coding.md#salesforce-mcp) is a great place to start for any Salesforce project, but there are also more general MCPs for code formatting, testing, and terminal commands that can be helpful.

## Consolidate your codebase 

The less code in your repository, the better. 
Try to scope your code to the feature you are trying to accomplish and not all the code and metadata in your org. 
This will help your AI coding assistants focus on the relevant code and provide better suggestions and feedback.

- Scratch orgs built with [CumulusCI (cci)](setup.md) can be a great way to create a clean and focused codebase for your project. 
- CCI also helps you shape your scratch orgs adjusting the settings in your org defaults, inserting sample data, applying permissions, setting up communities, setting picklist, and more.
- Consider using [scratch org snapshots](scratch-org-snapshots.md) supplemental functionality to large projects.

## Set up global instructions for your coding agent 

Global instructions give your coding agent context at the very beginning of each request you have for it. 
The instructions help the coding agent become a coder like you, understand your coding style and preferences, and provide better code suggestions and feedback.
The instructions can also highlight pitfalls to avoid and best practices to follow, which can help improve the quality of the code being generated 
and reduce the amount of time you spend on code reviews and debugging.

In your global instructions consider including:

- Code formatting rules (e.g. 2 spaces for indentation, single quotes for strings, etc.)
- Testing guidelines (e.g. write unit tests for all new code, use a specific testing framework, etc.)
- Terminal command best practices (e.g. always use `--dry-run` when running deployment commands, etc.)
- Where to put documentation (organize docs in a specific folder)
- Where to put project specific AI instructions (e.g. in a `AI_INSTRUCTIONS.md` file in the root of the repo)

See [GITHUB-COPILOT-INSTRUCTIONS.md](GITHUB-COPILOT-INSTRUCTIONS.md) for a fully built-out example of global instructions for Salesforce development with GitHub Copilot.

***Note:*** Global instructions should be project-agnostic and not include any specific context about your org, object names, field names, etc. That kind of context belongs in project-specific instructions (see next step).

***Note:*** Global instructions are a living document. As you work with your AI coding assistants, you may find that you need to update or add to your global instructions to improve the quality of the code being generated. Don’t be afraid to iterate on your instructions as you learn what works best for you and your project.

As you work with your AI observe what it is doing. When you see it struggling you can do the follow:

  - Wait until the AI succeeds and then ask a question like, "I see you were struggling and then figured out a solution, can we add this to our global instruction)"
  - Ask the AI agent if there are any [terminal tools or commands](vide-coding-terminal-add-ons.md) that would help it be more efficient in its work. If it identifies any tools that would be helpful, consider installing those tools and adding them to your global instructions.
  - If you find that the AI is consistently struggling with a particular type of task or code pattern, consider creating a specific MCP for that task or pattern to provide more targeted guidance to the AI

## Start with a user centered requirement document

Before you start asking your AI coding assistants to generate code, it can be helpful to create a user-centered requirement document that outlines the specific functionality you want to build. 
This document should be written in plain language and focus on the user’s needs and goals, rather than technical implementation details.

Consider putting the following documents in your docs folder (as defined in your global instructions):

- The user-centered requirement document that outlines the functionality you want to build.
- Any documentation related to the codebase. This can be things like:
  - A README file that explains the functionality of the code and how to use it.
  - Comments in the code that explain the purpose of different sections and any important details.
  - A user guide that provides step-by-step instructions for using the code.

Sometimes you will get your user requirements in word or a PDF. 
It is beneficial to convert those documents into markdown and put them in your repo so that your AI coding assistants can refer to them when generating code.
Here is a handy web based pdf to markdown converter: https://pdf2md.morethan.io/

Once you have this document:

- Present it to your AI coding assistants and ask them to generate code based on the requirements outlined in the document.
- Review the AI-generated requirements and tweak the document as needed to ensure that it accurately captures the functionality you want to build.

## Do some non-agentic coding

Sometimes you have a direction in mind of how you would like the project to go. 
It is helpful to start coding in that direction yourself to give your AI coding assistants a better understanding of the style and structure you are looking for.

Consider building out:

- The stubs of files and classes that you would like to see in the project (this can help with structure and naming conventions)
- Building Apex wrapper classes (this can help with understanding the data structures and patterns you want to use)
  - Wrapper classes are also very efficient for Lightning Web Components (LWC) development as your components make one efficient call for the data they need.

## Let the agent have at it... but keep the conversation going

At this point you have everything you need to let your coding agent start generating code for you. 
This is not the end of your work, but rather the beginning of a seasoned developer using understanding of code to direct the agent.

Things a human coder can do as your agent codes away:

- Watch the conversation the agent is having with itself for clues on where it is going and if it is struggling.
- Stop the agent at any time if you think it is going down the wrong path and you can give guidance to the approach.
- Review the code that the agent is generating and provide feedback to the agent on what it is doing well and what it can improve on (this may involve stopping the request).
- Let the agent do deploys if you have consolidated your codebase and are deploying to a safe org (e.g. a [scratch org](setup.md)). Deployment will highlight any errors and your agent can read and adapt to them.
- Ask the agent to build unit tests and run them to validate the code it is generating. If the tests are failing, ask the agent to read the error messages and fix the code until the tests are passing.
- Watch to make sure the agent is observing your [global instructions](GITHUB-COPILOT-INSTRUCTIONS.md) and if it is not, stop the request and remind it of the relevant instruction.

## When you think you are done, your not

You may have some really good working code that your agent produced and deployed for you, but that is not the end of the process.

Consider the following steps to wrap up your project:

- Ask your agent if there is anything it can do to make your code more efficient.
- If you have it installed, you can ask your agent to run [Salesforce Code Analyzer (SCA)](GITHUB-COPILOT-INSTRUCTIONS.md#salesforce-code-analyzer) on your code and fix any issues it identifies.
- Tell your agent to consider this code to be your initial release version and to build documentation around it. This can include things like:
  - A README file that explains the functionality of the code and how to use it.
  - Comments in the code that explain the purpose of different sections and any important details.
  - A user guide that provides step-by-step instructions for using the code.
- If your linter is showing warnings, ask your agent to fix those warnings and explain to you why those warnings were occurring and how it fixed them.