# Documentation Writing and Management Guidelines  

## Documentation Tools  
Most design documents use Markdown (*.md) format. Below is a list of tools used to write Markdown documents.  
| Tool               | Purpose                                  | Notes                   |  
| ------------------ | ---------------------------------------- | ----------------------- |  
| Visual Studio Code | Markdown, PlantUML editing, and PDF build |                         |  
| PlantUML           | Writing UML diagrams                     |                         |  
| yEd                | Creating various diagrams including UML  |                         |  

## Diagram Creation Methods  

**- Using PlantUML**  
There are two ways to insert PlantUML diagrams within Markdown:  
1. Create UML in a separate file (*.puml), export it as an image, and import the image into Markdown.  
2. Write the code directly within Markdown.  


**- Using yEd**  
yEd allows you to create diagrams by editing shapes and lines with a mouse, similar to PowerPoint, while PlantUML requires coding to create diagrams. Diagrams created with yEd can be exported in various image formats such as SVG, PNG, or JPG to be inserted into Markdown documents.  

### Document Structure  

The repository contains documentation as .md files corresponding to each directory in the `docs` folder, structured as follows:


```
XXXX.md
 - /graphml - .graphml files created with yEd  
 - /images - Images converted from diagrams created with yEd and PlantUML, such as PNG and SVG files  
 - /plantuml - .puml files created with PlantUML  

```
* Images inserted into the md files are exported with the same file names to manage the history of the files created with graphml and plantuml.  
This means that one image corresponds to one graphml or plantuml file.  

## Document Modification Example  

If changes occur in the `Software Architecture.md of docs/architecture`:  
 1. Write the content for 5.8 Notification Service  
 2. Add the yEd diagram for Notification Service (graphml/component_notification_service.graphml)  
 3. Export the .graphml to SVG and add it (images/component_notification_service.svg)  
 4. Write the corresponding content in the md file and insert the image

```
### 5.8. Notification Service  
Write a description of the notification service // Write content  
![](images/component_notification_service.svg) // Insert image  
```
<br>
