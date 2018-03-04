<img src="http://ocr-d.de/sites/default/files/Header1-Text-gold_3.png">

# Specification of the technical architecture, interface definitions and data exchange format(s)

## 1. Introduction
Lorem ipsum dolor sit amet

## 2. System architecture
The OCR-D project comprises of six overarching topics that are distinguished as separate modules. These modules are 
* Image Preprocessing, 
* Layout Recognition, 
* Text Optimization, 
* Model Training, 
* Long-term Preservation and Persistence, 
* Quality Assurance. 

Out of these six modules, the first four modules are directly responsible for processing of images in order to extract the required OCR features, while the fifth module focuses on the long-term archival, i.e., storage of digitized images, intermediary data, and final outcomes in a scientific data repository. 

A critical aspect of the OCR-D project is the orchestration of these modules in order to build systematic workflows that are not only flexible but also transparent and sustainable. An entire OCR-D workflow can be viewed in two perspectives. First, an abstract view, describing the interfaces on module-level with the input and output data, metadata, and parameters. Second, a detailed view describing the interfaces for each sub-module within each module. The detailed view is specific towards the realization of the module by the community and is subject to change based on the tools, technologies, and programming languages adopted by the community.  

Let's consider the Image Optimization module that has Cropping, Despeckling, Deskewing, Dewarping, and Binarization sub-modules. Based on the implementation approach, the community can have a monolithic implementation where all the submodules are tightly coupled, and only a single abstract module-level interface is exposed, or a loosely coupled implementation, where each submodule is exposed as an independent REST service.

## 3. Interface definitions
The interfaces, either module-level or sub-module have to be exposed via the REST specification. Regarding the metadata standard for modeling the OCR features, the [PAGE](https://github.com/PRImA-Research-Lab/PAGE-XML) format is adopted. For additional metadata such as bibliography metadata, description of the data, structure of the digitized documents, and other data relevant information, the [METS](https://www.loc.gov/standards/mets/) metadata standard is to be used. For this type of metadata, an OCR-D [METS](https://www.loc.gov/standards/mets/) profile has to be established with the accordance of the partners. It is expected that this [METS](https://www.loc.gov/standards/mets/)-profile will be composed of multiple other metadata standards such as [TEI](http://www.tei-c.org/index.xml), [textMD](https://www.loc.gov/standards/textMD/), [PREMIS](https://www.loc.gov/standards/premis/), [PAGE](https://github.com/PRImA-Research-Lab/PAGE-XML), etc.

In the following, we briefly describe the interfaces for the modules. These interfaces are described conforming to the REST specification.

Abstract view:  On the module level, there are four high-level REST services that correspond to each module. 

a.  Preprocessing: The preprocessing module comprises of two steps: (a) image characterization and (b) image optimization. In image characterization the metadata pertaining to intrinsic and extrinsic characteristic of an image are extracted and modeled as metadata and exported as a result for each image characterization. In image optimization, preprocessing steps Deskewing, Cropping, Binarization, Despecking, Dewarping. 

REST-Interface: documentImagePreProcessing    
HTTP Method: POST    
Input Data: [Mandatory] Image file + [Optional] Configuration file    
Input Parameters: List of module-specific parameters serialized in XML format    
Output Data/Metadata/Log: Preprocessed Image (Optimized Image)    
Log information: A minimum level of log information (template provided in Appendix)    
Module-output: Metadata describing the intrinsic and extrinsic characteristics    
HTTP Status Code: See response codes in Appendix    

b. Layout Analysis: The layout analysis module aims at extracting the logical structures within an image. This module contains few sub-modules such as Page Segmentation, Recognition of Text Lines, Segment Classification, and Document Analysis. Out of these four sub-modules, the Document Analysis sub-module is responsible for recording the metadata generated by the remaining sub-modules. For this purpose, here a standard interoperable format such as [METS](https://www.loc.gov/standards/mets/) is required. However, [METS](https://www.loc.gov/standards/mets/) is only a container format and a software/tool descriptive metadata model needs to be decided. The layout regions should be modeled in  [PAGE](https://github.com/PRImA-Research-Lab/PAGE-XML) that is further embedded within the [METS](https://www.loc.gov/standards/mets/). 

REST-Interface: documentImageLayoutAnalysis    
HTTP Method: POST    
Input Data: [Mandatory] Image file + [Optional] Configuration file    
Input Parameters: List of module-specific parameters serialized in XML format    
Output Data/Metadata/Log: [PAGE](https://github.com/PRImA-Research-Lab/PAGE-XML) containing the extracted layout regions    
Log information: See Appendix for the required minimum logging information    
HTTP Status Code: See response codes in Appendix    

c. Text Optimization: Text optimization can be automatic or semi-automatic requiring user interaction. For a semi-automatic solution a REST-interface is not possible, as dedicated software/tools with a graphical user interface is required. However, for function-based text optimization, wherein the optimization is completely automated (unsupervised post text correction), a REST-interface should be offered.

REST-Interface: documentTextOptimization    
HTTP Method: POST    
Input Data: [Mandatory] Segmented Text + [Optional] Configuration file     
Input Parameter: List of module-specific parameters serialized in XML format    
Output Data/Metadata/Log: Optimized text    
Output Metadata: (a) Service/tool/algorithm name and version, input parameters used, metadata describing the data, serialized in [TEI](http://www.tei-c.org/index.xml), [METS](https://www.loc.gov/standards/mets/) format. Potential candidate can also be [PAGE](https://github.com/PRImA-Research-Lab/PAGE-XML) with the UserDefined Attributes in WordType section.    
Log information: See Appendix for the required minimum logging information.    
HTTP Status Code: See response codes in Appendix    

d. Access to training infrastructures: Establishing ground truth is critical in OCR; the trained models are required to verify the output produced by the various OCR modules. Even though, generating the ground truth is highly dependent on the software/tools adopted by the community, the basic steps in generating the ground truth are always similar. Thus, for enabling automated, accessible, reusable, and simplified-workflows, the services/tools of the training infrastructure should be exposed as well-documented services. Moreover, the trained models should be stored and made accessible using a repository with a metadata schema (to be seen if such a metadata schema already exits or do we need to define a custom one) describing the model and its usage.  Additionally, the font used within the digitized image needs to be recognized using tools such as Matcherator, WhatTheFont, WhatFontIs, etc. The results need to be documented, for this metadata schema that can model this information is needed. For example, the [textMD](https://www.loc.gov/standards/textMD/) schema can be a potential candidate. Finally, the trained model needs to be stored in a model repository and made accessible for reuse. A sample REST interface is described below.

REST-Interface: generateTrainingModel    
HTTP Method: POST    
Input Data: [Mandatory] sample training data + [Optional] Configuration file     
Input Parameter: List of module-specific parameters serialized in XML format    
Output Data/Metadata/Log: Trained data model    
Metadata: description of the training steps, and quality of the trained data model from the basis of test data    
Log information: See Appendix for the required minimum logging information    
HTTP Status Code: See response codes in Appendix    

e. Auxiliary services: Auxiliary services represent the independent and reusable services that can be used by any module or sub-module. For designing clean workflows, the partner communities should avoid redundant implementation of functionality within their module, but instead provide this functionality as a reusable service in the Service Catalog. For example, a image format conversion service can be implemented using the [GraphicsMagick/ImageMagick](http://www.graphicsmagick.org/convert.html) library.

REST-Interface: convertImageFormat    
HTTP Method: POST    
Input Data: Image to be converted (accepted formats)    
Input Parameter: List of parameters required by the library.     
Output Data/Metadata/Log: Converted image    
Log information: See Appendix for the required minimum logging information.    
HTTP Status Code: See response codes in Appendix    

f. Services from the OCR-D repository: For enabling systematic execution of OCR workflow with the necessary long-term persistence of data in the repository, modeling, storing, and making metadata searchable in the metadata storage, there needs to be a well-defined set of services that have to be made available. 

Metadata storage service: The metadata storage service offers a single point of storage and access to the metadata generated through the entire OCR workflow. This metadata can be static or dynamic. In terms of static metadata, the various metadata standards/models used during the OCR workflow have to be systematically modeled in the [METS](https://www.loc.gov/standards/mets/) format conforming to the OCR-D [METS](https://www.loc.gov/standards/mets/)-profile. For example, [PREMIS](https://www.loc.gov/standards/premis/) standard for data/metadata preservation, and [TEI](http://www.tei-c.org/index.xml) are to be used as per their application and finally will be modeled within the [METS](https://www.loc.gov/standards/mets/) profile. 

Data storage service: In principle, the service stack of the KIT Data Manager exposes this data storage service. The minimalistic functionality of this service is to allow communities to store and access their data from the repository. 

Services registry and catalog: For cataloguing the release version services, a service registry needs to be setup for the all the partner communities. As we have decided to adopt the Taverna WfMS, the [ServiceCatalographer](https://github.com/myGrid/ServiceCatalographer) project that provides the possibility to register REST and SOAP services can be setup for out use.

Workflow registry ([myexperiment.org](https://www.myexperiment.org/)): With the services registered in a dedicated registry like ServiceCatalographer, the next step is to register the entire workflows in a repository. The [myexperiment.org](https://www.myexperiment.org/) environment provides the appropriate workflow repository for storing and accessing the workflows.

## 4. Data and metadata exchange format(s)
* [METS](https://www.loc.gov/standards/mets/)
* [PAGE](https://github.com/PRImA-Research-Lab/PAGE-XML)
* [ALTO](https://www.loc.gov/standards/alto/)

## 5. Repository
TODO

## 6. Appendix
Minimal log information that each module should provide is described below.  Any additional, important information pertaining to the execution of the services within the module should be appended to this information. The logging information is necessary for detecting failed services, tracing the execution, and debugging purposes. The information required for reproducing the results will be extracted from these logs. 

Service name: <name of the module/sub-module>    
Service version: <module version>    
Service description: <textual description of the service>    
Service URL: <location where the service is deployed>    
Service Implementation: <implementation library name, version>    
Input data URL: <URLs of the input data>    
Input parameters: <list of input parameters>     
Output file name and size: <name of the output file and size>     
Exit code: <Enumeration of Exit codes, 0 = OK, any value > 0 corresponds to a particular error code>    
Service invocation Timestamp: <timestamp>    
Service Processing Time: <time in milliseconds>    
  
Mapping of Exit Codes to HTTP Responses

| Exit Code | HTTP Response |
| --------- | ------------- |
| 0 | 200 (Success) |
| 1 | 400 (Bad formed request) |
| 2 | 500 (General server error) |
| 3 | 503 (Service unavailable) |
| ... | ... |
