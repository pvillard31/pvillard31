# Getting ready for Apache NiFi 2.0

Apache NiFi 2.0 is around the corner and it is going to bring a lot of new features that I, personally, am very excited about. We can mention key features such as:

- Making Python a first class citizen in NiFi, allowing Python aficionados to easily and quickly build NiFi components in Python. This will bring closer the data engineers and the data scientists communities as well as greatly expanding the set of use cases that NiFi can cover.
- Giving the possibility to run a Process Group using NiFi Stateless. NiFi Stateless has been around for quite some time but had never been easy to use until NiFi 2.0 where you can enable it at process group level. This gives the opportunity to use NiFi for critical use cases where a flow should be considered as a transaction (think CDC for example).
- Rules Engine to provide feedback to the flow designers with regard to best practices, to give recommendations for the configuration of the components, etc.

However this major new release of NiFi comes with breaking changes and you can take actions starting today to be in the best possible spot for when you want to upgrade to NiFi 2.0. The below list is not exhaustive but gives a good starting point. You can find out more about the NiFi 2.0 goals by going [here](https://cwiki.apache.org/confluence/display/NIFI/NiFi+2.0+Release+Goals).

Note that the community is planning to build some tooling to automate as many of those actions as possible but it’s for the best if you go through this list and take actions on your side. While there will be tooling to help, there could always be some edge cases we can’t cover / anticipate.

## Java 21

NiFi 2.0 will only support Java 21 so you want to make sure you have Java 21 installed on your NiFi nodes before upgrading. Note that you can already use the latest 1.x releases of NiFi with Java 21.

## Templates

The concept of templates is going away. The XML templates are stored in memory in NiFi as well as in the persisted flow definition (flow.xml.gz & flow.json.gz files) and it caused a lot of problems with some of our biggest NiFi users when they had tens or hundreds of massive templates (with thousands of components). Removing all of this will bring a lot more stability to NiFi and improve memory usage.

If you have templates, you will want to export those templates as JSON definitions or version the templates into a NiFi Registry instance. The best practice is really to use a NiFi Registry in combination with NiFi when it comes to version control and share / reuse flow definitions.

To do that:
- If your template is a process group, you can just drag and drop the template on the canvas and then right click on it and export it as a flow definition (JSON file) or start version control in your NiFi Registry if you have one configured.
- If your template is not a process group but directly a flow with components, you’d want to drag and drop a process group, then go into that process group and drag and drop your template there. You can then go back to the parent process group containing your template and export it as a flow definition or start version control on it.

## Variables

Variables and Variable Registry are going away. It was coming with a lot of limitations such as the need for expression language support in a property to reference a variable, and the impossibility to store sensitive values. Parameter Contexts have been around for a while now and have been improved over the last few years. For example, we recently added the concept of Parameter Context Provider to source the value of parameters from external stores (like HashiCorp, Vaults of the cloud providers, etc).

Make sure to spend some time moving from variables to parameters. This is most likely the most impacting change that will require some rework on the flows. It’s a good opportunity to think about the right approach and how to split parameters into multiple Parameter Contexts and use inheritance between the contexts when you want to share the same parameters across multiple use cases.

## Event Driven Scheduling Strategy

The Event Driven Scheduling Strategy was an option available on some processors. This was an experimental feature in NiFi and didn’t prove to bring any significant performance improvements. This feature is going away with NiFi 2.0.

If you have components configured with this scheduling strategy (you can find those using the search bar in NiFi by typing: event), and update those components to use the “Timer Driven” scheduling strategy instead.

## Removed Components

We’re using NiFi 2.0 as an opportunity to remove a lot of components that were deprecated and for which better alternatives are available. The exhaustive list can be found [here](https://cwiki.apache.org/confluence/display/NIFI/Deprecated+Components+and+Features) and the community is also providing migration steps [here](https://cwiki.apache.org/confluence/display/NIFI/Migrating+Deprecated+Components+and+Features+for+2.0.0). If your flows are using some of those processors, you probably want to start using the alternatives to make the upgrade seamless. Otherwise the components would become “ghost components” when upgrading to the new version of NiFi.

Note: if you’re running a recent version of NiFi, a dedicated log file has been added: nifi-deprecation.log. It can be a good place to review for runtime usage of features that are targeted for removal. More details can be found [here](https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#deprecation-logging).

## Custom Components

If you have built custom components, you’d likely want to update the dependencies in your components to reference the NiFi 2.0 APIs and rebuild the components with a newer version to make sure they’re working properly after the upgrade. A command has been added to the CLI toolkit in NiFi to recursively change the version of all instances of a component to a newer version.

## Conclusion

There is a lot of great things coming with NiFi 2.0 and you don’t want to miss any of these. Be ready and start planning for the upgrade for when NiFi 2.0 comes out. I hope this overview has been helpful and you can always reach out to the NiFi community via the mailing lists or on Slack!
