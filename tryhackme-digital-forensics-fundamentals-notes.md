# TryHackMe Digital Forensics Fundamentals

## Introduction to Digital Forensics

Digital forensics applies structured methods to identify, acquire, examine, analyze, and report on digital evidence. It can support investigations of cybercrime, but it is also relevant to traditional crimes when digital devices or digital records may help establish what happened. A phone, laptop, storage device, photograph, video, document, or digital map can contain evidence relevant to an investigation.

The room uses a bank robbery scenario to show that digital evidence is not limited to crimes committed online. A physical crime may leave digital traces that help investigators understand events, relationships, locations, or timelines. The important point is that digital forensics concerns the handling and interpretation of digital evidence, regardless of whether the original crime was committed through a computer.

## Digital Forensic Methodology

A common digital forensic methodology has four phases: collection, examination, analysis, and reporting. Different frameworks may use slightly different names or divide the work differently, but the general process is consistent. Investigators acquire relevant evidence, examine it to identify useful information, analyze that information in context, and document their findings.

Collection involves obtaining digital evidence through an authorized and appropriate process. Examination involves extracting and identifying information of interest from the acquired evidence. Analysis involves correlating the information, considering its relevance, and drawing conclusions supported by the evidence. Reporting documents the methods used, the observations made, and the conclusions that can reasonably be drawn. In a legal investigation, reporting may include presenting findings in court and explaining the methods as an expert witness.

The room references the National Institute of Standards and Technology, commonly known as NIST, as a source of digital forensic guidance. Some practitioners combine examination and analysis into one broader analysis stage for simplicity. When answering questions about a particular framework, however, I need to follow the phase names used by that framework. In the four-phase model taught in this room, extracting data of interest is examination, while correlating data to draw conclusions is analysis.

Digital forensics includes several related areas. Disk forensics focuses on data stored on storage media. Memory forensics examines data held in RAM. Network forensics studies network traffic and related records. Mobile forensics examines phones and other mobile devices. Malware forensics analyzes malicious software and its behavior. Cloud forensics concerns evidence held in cloud services and infrastructure. These areas use related forensic principles, but the acquisition methods and technical details differ.

## Evidence Acquisition and Integrity

Evidence acquisition must be conducted with proper authorization. Investigators need an appropriate legal basis or permission to examine a device or account. Having access to a device does not automatically mean that every type of examination is authorized.

The integrity of digital evidence must be preserved throughout collection, transport, storage, examination, and reporting. Digital evidence can be changed accidentally as well as deliberately, so forensic procedures are designed to reduce the risk of altering the original evidence and to make any handling traceable.

A chain-of-custody document records the history of evidence. It typically identifies the evidence, who collected it, when and where it was collected, how and where it was stored, and who accessed or transferred it and when. This record helps demonstrate how the evidence was handled and whether there are unexplained gaps in its custody.

A write blocker is used during storage-device acquisition to prevent writes to the source media while allowing data to be read. This helps reduce the risk that the examination system changes files or metadata on the original drive. Write blockers can be hardware-based or software-based, and the appropriate method depends on the procedure, equipment, and requirements of the case. A write blocker does not by itself guarantee that every aspect of evidence handling is correct, so acquisition and verification procedures still matter.

The room asks which tool is used to help ensure data integrity during collection. Its expected answer is a write blocker. It also asks for the document that records details about collected evidence. That document is the chain of custody.

## Windows Forensics: Disk and Memory Images

A forensic image is a copy acquired for forensic examination. A disk image captures data from storage media, such as a hard disk drive or solid-state drive. Storage is generally non-volatile, meaning that the stored data normally remains available when the computer is powered off. A disk image may contain documents, file-system information, browser artifacts, application data, and other records, depending on what was present and what the acquisition captured.

A memory image captures data from a computer's random-access memory, or RAM. RAM is volatile, meaning that its contents are normally lost when power is removed or the computer restarts. Memory can contain information about the system's state at the time of acquisition, including running processes, open files, and current network connections. Because this information can disappear, investigators consider volatility when deciding the order in which evidence should be collected.

The order of volatility is a way of prioritizing evidence that is likely to disappear or change quickly. Volatile information, such as RAM contents and some live system state, often needs to be collected before less volatile information stored on a drive. The exact order depends on the circumstances and the investigative procedure. Investigators must also consider safety, authorization, system stability, and whether interacting with a live system could change evidence.

The room introduces several tools used in forensic workflows. FTK Imager is presented as a tool for acquiring forensic images, particularly in Windows environments. Autopsy is an open-source digital forensics platform used to examine disk images and review artifacts such as file metadata, search results, and recoverable deleted files. DumpIt is presented as a tool for acquiring memory from a Windows system. Volatility is used to analyze memory images and can help investigate processes, connections, and other artifacts present in memory.

These tools have different purposes. Acquiring an image and analyzing an image are separate tasks, and the choice of tool depends on the evidence type and operating system. The room focuses on introductory tools, while professional environments may also use commercial forensic products. The key distinction in the task is that a memory image is used to collect volatile data.

## Metadata and File Examination

Metadata is information about a file or other data. Depending on the file type, metadata can include an author, creation date, modification date, software used to create the file, dimensions, device details, or location information. Metadata can help establish context and guide an investigation, but it must be interpreted carefully. A metadata field is not automatically proof that an event happened at a particular time or that a particular person created a file.

The room demonstrates `pdfinfo`, a command-line utility that displays metadata and properties of a PDF document. Depending on the file, the output may include the author, producer, creation date, modification date, number of pages, PDF version, and whether the document contains JavaScript. The general command structure is:

```bash
pdfinfo filename.pdf
```

The `man` command can display the manual for a command-line tool when its documentation is installed. For example:

```bash
man pdfinfo
```

The room uses the file `ransom letter.pdf` and identifies the author field as `Ann Gree Shepard`. This is the value observed in the provided training material.

## Image Metadata and EXIF

EXIF stands for Exchangeable Image File Format. It is a standard for storing metadata in image files produced by many cameras and phones. Depending on the device and its settings, EXIF metadata may include the camera model, capture date and time, camera settings, and GPS coordinates. Some platforms remove or alter metadata when images are shared, so not every image contains all of these fields.

The room introduces ExifTool, a command-line utility for reading metadata from image files. Its basic command structure is:

```bash
exiftool filename.jpeg
```

The output can contain many fields, so the useful approach is to identify the specific field required for the investigation rather than assume every field is relevant. For example, GPS latitude and GPS longitude may help locate where an image was recorded, while the camera model field may identify the device model reported in the metadata.

When coordinates are used in a mapping service, they need to be copied and formatted correctly. Latitude and longitude values may include degrees, minutes, seconds, or directional indicators, depending on how the metadata is displayed. I should preserve the coordinate information and convert it carefully rather than remove symbols or characters without understanding their meaning.

In the practical exercise, the image metadata contained GPS coordinates that led to Milk Street in London, United Kingdom. The camera model field identified the camera as a Canon EOS R6. These are the results reported for the room's provided image, not general conclusions about other images.

## Practical Exercise

The practical task provided a set of files for examination. I used the terminal to navigate to the exercise directory and list the available files. The task then used metadata tools to answer questions about a PDF document and a JPEG image.

For the PDF question, I ran `pdfinfo` against `ransom letter.pdf` and inspected the author field. The value shown in the room was `Ann Gree Shepard`.

For the image questions, I ran ExifTool against `letter image.jpeg` and examined the metadata output. The GPS latitude and longitude were used to locate the image in Google Maps, which led to Milk Street. I then checked the camera model field and found `Canon EOS R6`.

These steps illustrate a basic forensic workflow: identify the supplied evidence, use a tool appropriate to the file type, inspect the relevant metadata, and record the result. Metadata can provide useful investigative leads, but any conclusion should consider whether the metadata is complete, trustworthy, and consistent with other evidence.

## Conclusion

This room introduced digital forensics as a structured process for handling digital evidence in both cybercrime and traditional investigations. It covered the collection, examination, analysis, and reporting phases; the importance of authorization, chain of custody, and write blockers; the difference between disk and memory images; and introductory tools for image acquisition and examination.

The practical exercise demonstrated how `pdfinfo` can reveal PDF metadata and how ExifTool can reveal image metadata such as GPS coordinates and camera model information. The main lesson is that digital forensic findings depend not only on the tools used, but also on careful evidence handling, preservation of integrity, and conclusions that remain within what the evidence actually supports.
