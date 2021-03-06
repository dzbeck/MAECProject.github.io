---
layout: flat
title: Capturing AV Classification Results
summary: This Idiom describes the process of capturing the classifications as reported by anti-virus (AV) tools when executed against a particular malware instance.
---

This Idiom describes the process of capturing the classifications as reported by anti-virus (AV) tools when executed against a particular malware instance. As with all analysis-derived results, those that come from AV tools can be captured through the use of a MAEC Bundle. However, such output will be captured exclusively through the use of the [AV Classification](/data-model/{{site.current_version}}/maecBundle/AVClassificationType) entity.

## Scenario

In this scenario, a malicious PE binary has been scanned with a number of anti-virus (AV) engines, which provide detection and associated classification information for the binary. For this example, we'll assume that the binary was scanned with engines from Microsoft, Symantec, and Trend Micro.

## Data model

The following are the important MAEC data model constructs used in this idiom:

* [AV Classifications](/data-model/{{site.current_version}}/maecBundle/AVClassificationsType): the AV Classifications entity is used to capture one or more AV classifications for a malware instance, and is captured in a [MAEC Bundle](/data-model/{{site.current_version}}/maecBundle/BundleType).

* [AV Classification](/data-model/{{site.current_version}}/maecBundle/AVClassificationType): the AV Classification entity captures a single AV Classification as reported by an anti-virus tool, and is a MAEC extension of the [CybOX ToolInformationType](/data-model/{{site.current_version}}/cyboxCommon/ToolInformationType)

Let's explore the [AV Classification](/data-model/{{site.current_version}}/maecBundle/AVClassificationType) entity in more detail - it contains the following fields that are most relevant to the capture of anti-virus classification results:

1.	*Vendor*. The name of the AV tool vendor, e.g. Microsoft.

2.	*Version*. The version of the AV tool, if available.

3.	*Engine_Version*. The version of the AV engine used by the AV tool that assigned the classification to the malware instance. 

4.	*Definition_Version*. The version of the AV definitions used by the AV tool that assigned the classification to the malware instance.

5.	*Classification_Name*. The classification assigned to the malware instance object by the AV tool, if one was given. E.g., "Zbot.123". Note that if the AV tool does not detect or provide a classification for the malware instance, this field should not be included.

## Process

As with many of the other Idioms, the first step is to create a [MAEC Package](/data-model/{{site.current_version}}/maecPackage/PackageType) with a [Malware Subject](/data-model/{{site.current_version}}/maecPackage/MalwareSubjectType) for capturing the information about the malware instance being analyzed. We should also add an [Analysis](/data-model/{{site.current_version}}/maecPackage/AnalysisType) entity to the Malware Subject to capture some details relating the particular analysis that we're performing. The information on this process is not covered in this idiom, but can be found in the corresponding [Creating a MAEC Package](../package_creation) and [Capturing Analysis Metadata](../analysis_metadata) idioms.

Next, a [MAEC Bundle](/data-model/{{site.current_version}}/maecBundle/BundleType) is created. Once created, we must set the "content_type" attribute on the Bundle to define the type of content that it is characterizing.  In this case, since we're capturing the output of AV engines, we should set it to a value of "static analysis tool output" since most such engines statically analyze binaries to provide their classification. This is one of the values contained in the BundleContentTypeEnum enumeration used by this field. Finally, we should set the defined_subject attribute on the Bundle to a value of "false", since this Bundle will be contained in a Malware Subject, which has already defined the particular malware instance being characterized.

Now that we've set up the Bundle that will capture the AV tool classification output, we can begin to populate it with these results. For the sake of simplicity in our example, let's assume that we only know the name of the vendor of each AV tool, along with the particular classification that it assigned to the PE binary (this assumes that each tool was able to detect the binary). Thus, we will create three instances of the the [AV Classification](/data-model/{{site.current_version}}/maecBundle/AVClassificationType) entity, one each for Microsoft, Symantec, and Trend Micro. In each such instance, we will use the "Vendor" field to capture the name of the tool vendor, and also the "Classification_Name" field to capture the actual classification that it gave to the PE binary.

After creating these [AV Classification](/data-model/{{site.current_version}}/maecBundle/AVClassificationType) instances, the remaining task is to add them to the [Bundle](/data-model/{{site.current_version}}/maecBundle/BundleType) that was previously created. To do this, we simply need to use the [AV_Classifications](/data-model/{{site.current_version}}/maecBundle/AVClassificationsType) field at the root level of the Bundle, to which we'll add the three [AV Classification](/data-model/{{site.current_version}}/maecBundle/AVClassificationType) instances.

With the [Bundle](/data-model/{{site.current_version}}/maecBundle/BundleType) populated with the results of the AV tools, the final step is to add it to the Malware Subject. To do, we'll use the [Findings_Bundles](/data-model/{{site.current_version}}/maecPackage/FindingsBundleListType) field, and specifically will populate its child "Bundle" field with the Bundle that we've constructed.

## XML

{% highlight xml linenos %}
<maecPackage:Bundle defined_subject="false" id="example:bundle-c374ea27-8520-4ab7-9fba-3f18050d4e1c" schema_version="4.1" content_type="static analysis tool output">
	<maecBundle:AV_Classifications>
	 <maecBundle:AV_Classification>
	  <cyboxCommon:Name>Microsoft</cyboxCommon:Name>
	  <maecBundle:Classification_Name>PWS:Win32/Zbot.gen!B</maecBundle:Classification_Name>
	 </maecBundle:AV_Classification>
	 <maecBundle:AV_Classification>
	  <cyboxCommon:Name>Symantec</cyboxCommon:Name>
	  <maecBundle:Classification_Name>Backdoor.Paproxy</maecBundle:Classification_Name>
	 </maecBundle:AV_Classification>
	 <maecBundle:AV_Classification>
	  <cyboxCommon:Name>TrendMicro</cyboxCommon:Name>
	  <maecBundle:Classification_Name>TSPY_ZBOT.TD</maecBundle:Classification_Name>
	 </maecBundle:AV_Classification>
	</maecBundle:AV_Classifications>
</maecPackage:Bundle>
{% endhighlight %}

[Full XML](maec_av_classification.xml)

## Python

{% highlight python linenos %}
# Create the AV Classifications
av1 = AVClassification()
av1.name = "Microsoft"
av1.classification_name = "PWS:Win32/Zbot.gen!B"
av2 = AVClassification()
av2.name = "Symantec"
av2.classification_name = "Backdoor.Paproxy"
av3 = AVClassification()
av3.name = "TrendMicro"
av3.classification_name = "TSPY_ZBOT.TD"

# Add the AV classifications to the Bundle
b.add_av_classification(av1)
b.add_av_classification(av2)
b.add_av_classification(av3)
{% endhighlight %}

[Full Python](maec_av_classification.py)

## Further Reading
* [Creating a MAEC Bundle](../bundle_creation)
* [Creating a MAEC Package](../package_creation)
* [Capturing Static Analysis Results](../static_analysis)