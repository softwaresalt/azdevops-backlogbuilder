# Azure DevOps Product Backlog Accelerator

## Overview

The Product Backlog Accelerator is a project that demonstrates how to leverage **GitHub Copilot** and AI-powered development practices to synthesize various project artifacts into well-structured Agile features and user stories. This project focuses on the ACME Predictive Maintenance project, but the methodology can be applied to any software development initiative.

## Purpose

This repository showcases how to transform disparate project inputs—including workshop transcripts, architecture diagrams, and presentation materials—into a comprehensive product backlog that follows Agile best practices.

## Input Sources

The project synthesizes a variety of artifact types (e.g., workshop transcripts, architecture diagrams, presentation decks) into a structured backlog. Previously referenced sample transcript, diagram, and deck files have been removed from the repository to reduce size. You can supply your own source materials and follow the same workflow.

## Output Artifacts

The accelerator produces structured Agile artifacts in markdown format:

### 📋 Product Backlogs

- `backlog-md/product-backlog-gem.md` - Backlog with task breakdown (Gemini 2.5 Pro output).
- `backlog-md/product-backlog-gpt5.md` - Feature/user story focused backlog (GPT-5 output).

### 📄 Work Item Documentation

- `review-doc/WorkItemsFromWorkshops.docx` - Review-ready backlog document (live-edit capable)
- `backlog-yml/product-backlog-tasklevel.yaml` - Structured YAML format for tool integration (task-level granularity)

## How to Use GitHub Copilot for Synthesis

### Step 1: Context Preparation

1. **Open the Workspace**: Load the `backlogbuilder.code-workspace` in VS Code
2. **Enable Copilot**: Ensure GitHub Copilot is active and authenticated
3. **Load Context**: Open key files to provide Copilot with context about the project

### Step 2: Transcript Analysis

Use GitHub Copilot to analyze workshop transcripts:

```markdown
## Prompt Example:
"Analyze the following workshop transcript and extract key requirements, 
technical decisions, and business needs. Focus on identifying:
- Functional requirements
- Non-functional requirements (performance, security, availability)
- Technical constraints
- Business drivers
- Stakeholder concerns"
```

**Copilot Techniques:**

- Use `/doc` command to generate documentation from transcript content
- Leverage inline chat to ask specific questions about transcript sections
- Use `Ctrl+I` to generate summaries of long transcript sections

### Step 3: Architecture Diagram Interpretation

While Copilot cannot directly read images, you can:

1. **Describe Diagrams**: Create textual descriptions of architecture diagrams
2. **Use Image Context**: Reference diagram filenames and descriptions in your prompts
3. **Cross-Reference**: Link diagram elements to transcript discussions

```markdown
## Prompt Example:
"Based on the AzureTenant.png and OnPremSystem.png architecture diagrams, 
and the workshop transcripts discussing hybrid cloud architecture, 
generate user stories for the data flow between on-premises SCADA systems 
and cloud services."
```

### Step 4: Feature and User Story Generation

Use Copilot to transform requirements into Agile features:

```markdown
## Prompt Example:
Based on the workshop transcripts, the system context diagram, and system recommendations, create a product backlog composed of Features, User Stories designed to break down the implementation of Acme-Sub's future state system down to granular detail.
- The backlog must incorporate a key direction that HiveMQ will be sending messages directly to Event Hub and will not have a cloud instance.
- Each Feature must include a Title and Description.  The format should be Feature: {title text}
- Each User Story must include a Title, Description, and Acceptance Criteria; Start the Description section of the user story with the As a user persona phrasing; provide a short descriptive title.  The format should be User Story: {title text}
- Use a standard Agile template for building features and user stories.
- Do not create epics; all features will be for a single epic to be created outside of this document.
- Format the document in markdown.
- Features should have heading 2 formatting.
- User Stories should have heading 3 formatting.
- Name the file product-backlog.md

## User Story Template:
**As a** [persona] 
**I want** [goal]
**So that** [business value]

**Acceptance Criteria:**
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

**Definition of Done:**
- [ ] Code complete and reviewed
- [ ] Unit tests written and passing
- [ ] Documentation updated
- [ ] Deployed to test environment
```

**Copilot Chat Commands:**

- `@workspace /explain` - Understand existing backlog structure
- `@workspace /fix` - Improve existing user story formatting
- `@workspace /new` - Generate new features based on context

### Step 5: Backlog Refinement

Refine the backlog with additional context and online research:

```markdown
Revise this product backlog: 
- Identify where user stories could be broken down into smaller stories and break out as separate user stories.
- Provide more detail in each user story to give the user more context.
- Research online sources for best practices, practical advice, and standard capabilities; incorporate research into features and user stories.
- Remove feature/user story [X] since this is out of scope.
```

### Step 6: Further Backlog Refinement

Use iterative prompting to refine the backlog:

1. **Size Estimation**: Ask Copilot to estimate story points
2. **Dependency Analysis**: Identify story dependencies
3. **Sprint Planning**: Group stories into logical sprints
4. **Risk Assessment**: Identify technical and business risks

### Step 7: Review Current Backlog with Team

Once you have iterated on the backlog and have what you feel is a viable initial draft of the backlog for your customer project, copy/paste the HTML output of the markdown into a Word document.

In your Teams share for the project, upload your initial draft and review the backlog with the team to get feedback from all of your stakeholders to ensure that the approach is aligned with each stakeholder's perspective on their area of concern.  Note that a key value of doing this way is the live edit capabilities of Word to get everyone's input at once.

Once you have a viable backlog from your review process, you can convert the doc back into markdown with the following tool:

```code-block
py -m pip install markitdown[all]

markitdown WorkItemsFromWorkshops.docx -x docx -o product-backlog-reviewed.md
```

### Step 8: Generate Tasks (optional)

```markdown
### Prompt Example:
Instructions:  
- For each user story that does NOT have any tasks under it, create Agile tasks to cover each of the expected outcomes from the user story.
- Each Task must include a Title and Description; the description should include granular - implementation directions and details based on documented best practices and online documentation.
- Elaborate the Agile task description to include Technical Details and Expected Outcomes, a Definition of Done, and a list of technical Dependencies (not dependencies on other user stories or tasks).
- Use a relatively flat tone: minimize use of adjectives and superlatives. Avoid emphatic words such as easily or readily.
- Research online sources for best practices, practical advice, and standard capabilities; incorporate results into task breakout and task descriptions.
- Insert the new tasks under the user story.  Apply Heading 4 to each Task entry.
```

### Step 9: Review Tasks and Convert to YAML

Review the tasks to make sure they make sense. You may want to remove or combine some tasks. User stories may also need to be reassigned under different features based on area of concern (step 7).
Once you have a full backlog at the user story level or the task level, you can convert a task-level markdown document (your own file) to YAML so that it can be imported into your ADO project.

Use the [azdevops-workitem-loader](https://github.com/softwaresalt/azdevops-workitem-loader) project to load your backlog to ADO. The repository already includes an example YAML file at `backlog-yml/product-backlog-tasklevel.yaml`.

```markdown
### Prompt Example
Instructions:
- Restructure the contents of <your-task-level-markdown-file>.md as a YAML file.
- Feature nodes should have a Title and Description.
- User Story nodes under the feature should be indented as children of the feature.
- User Story nodes should include Title, Description, Acceptance Criteria.
- Task nodes under the User Story nodes should be indented as children of the User Story.
- Task nodes should include Title and Description. The description may include Description, Technical Details, Expected Outcomes, Definition of Done, and Dependencies.
- Preserve original markdown formatting inside description and acceptance criteria fields.
```

## Best Practices

### 🎯 Effective Prompting

1. **Be Specific**: Provide clear context about the project domain
2. **Use Examples**: Reference existing good examples in the workspace
3. **Iterate**: Refine outputs through follow-up questions
4. **Cross-Reference**: Connect information across multiple sources

### 📚 Context Management

1. **Keep Files Open**: Maintain relevant files in VS Code tabs
2. **Use Workspace Search**: Leverage `@workspace` to reference project content
3. **Maintain Consistency**: Ensure terminology consistency across artifacts
4. **Version Control**: Track changes to understand evolution

### 🔄 Quality Assurance

1. **Review Generated Content**: Always validate AI-generated requirements
2. **Stakeholder Validation**: Confirm outputs with project stakeholders
3. **Technical Feasibility**: Ensure technical accuracy of generated stories
4. **Business Alignment**: Verify business value statements are meaningful

## Generating Copilot Instructions

To maximize GitHub Copilot's effectiveness, you should create a comprehensive `copilot-instructions.md` file that provides context about your project. This file helps Copilot understand your domain, architecture, and specific requirements.

### Step 1: Analyze Project Context

Use GitHub Copilot to analyze your workspace and generate appropriate instructions:

```markdown
## Prompt for Copilot Chat:
"@workspace analyze all the files in this project and generate comprehensive 
copilot-instructions.md content that includes:

1. Project overview and business context
2. Key architectural principles and constraints
3. Technology stack and frameworks used
4. Domain-specific terminology and concepts
5. Coding standards and best practices
6. Common patterns and anti-patterns
7. Security and compliance requirements
8. Integration points and dependencies

Focus on information that would help an AI assistant understand the project 
context when generating code, documentation, or making recommendations."
```

### Step 2: Create the Instructions File

1. **Create the directory structure**:

   ```bash
   mkdir -p .github
   ```

2. **Generate initial content**: Use Copilot Chat with the above prompt

3. **Refine with project specifics**: Add details from your transcripts and diagrams

### Step 3: Structure Your Instructions

Your `copilot-instructions.md` should include these sections:

#### Project Overview

- Business purpose and goals
- Target users and stakeholders
- Success criteria

#### Architecture & Design Principles

- System architecture (from your diagrams)
- Key design constraints
- Non-functional requirements

#### Technology Stack

- Programming languages and frameworks
- Cloud services and infrastructure
- Development tools and platforms

#### Domain Knowledge

- Business terminology and concepts
- Industry-specific requirements
- Regulatory or compliance needs

#### Development Guidelines

- Coding standards and conventions
- Testing requirements
- Documentation expectations
- Security best practices

### Step 4: Workspace-Specific Instructions

For this Product Backlog Accelerator project, include:

```markdown
## Example Copilot Instructions Generation:
"Based on the ACME Predictive Maintenance project context, generate 
copilot-instructions.md that includes:

- Hybrid cloud/on-premises architecture principles
- Manufacturing and IoT domain knowledge
- Microsoft Fabric and services context
- Agile methodology and user story formatting
- MQTT, SCADA, and industrial automation concepts
- Security requirements for industrial environments
- Multi-tenant considerations"
```

### Step 5: Maintain and Update

- **Regular Reviews**: Update instructions as the project evolves
- **Team Feedback**: Incorporate feedback from developers using Copilot
- **Version Control**: Track changes to understand what works best
- **Testing**: Validate that instructions improve Copilot suggestions

### Best Practices for Copilot Instructions

1. **Be Specific**: Include exact terminology and concepts from your domain
2. **Provide Examples**: Show patterns and anti-patterns with code examples
3. **Context Hierarchy**: Structure from general to specific concepts
4. **Update Regularly**: Keep instructions current with project changes
5. **Team Alignment**: Ensure all team members understand and follow the instructions

## Getting Started

1. **Install GitHub Copilot**
   - Install the GitHub Copilot extension
   - Sign in with your GitHub account
   - Ensure Copilot is active

1. **Load Context**
   - Open key transcript files
   - Review existing backlog examples
   - Include copilot instructions in `.github/copilot-instructions.md`
   - If using Gemini Code Assist, include instructions in `gemini.md`

1. **Start Synthesizing**
   - Use Copilot Chat to analyze transcripts
   - Generate features and user stories
   - Refine outputs iteratively

## Tips for Success

- **Start Small**: Begin with one transcript or diagram
- **Build Context**: Gradually add more sources to your analysis
- **Validate Early**: Check outputs with stakeholders frequently
- **Iterate Often**: Use feedback to improve subsequent generations
- **Document Assumptions**: Record decisions and assumptions made during synthesis
- **Maintain Traceability**: Link generated items back to source materials

## Contributing

To contribute to this accelerator:

1. Follow the established patterns for new artifacts
2. Update this README when adding new synthesis techniques
3. Include clear examples of prompts that work well
4. Document any project-specific context or constraints

## License

This project is available under the [MIT License](LICENSE).

---

*This README was created as part of demonstrating AI-powered development practices using GitHub Copilot for requirements synthesis and Agile backlog creation.*
