# Migration and End-User Services

## Plan a migration around the target outcome

Choose a migration path from the desired business result, application constraints, and acceptable change:

- **Rehost** moves a workload with limited application changes.
- **Replatform** makes targeted changes to use a managed platform or improve operations without redesigning the full application.
- **Refactor or re-architect** changes the application to meet goals that a direct move cannot satisfy.
- Some applications should instead be replaced with a product, retired, or retained temporarily. A migration plan should allow those outcomes rather than treating every server as a lift-and-shift project.

Discovery and execution services play different roles. AWS Application Discovery Service helps collect information about an existing environment; Migration Hub can provide a place to track migration progress; Application Migration Service supports rehosting eligible servers; Database Migration Service supports database migration and replication; VM Import/Export can import or export supported VM images. These tools do not remove the need to assess dependencies, licensing, application compatibility, data transfer, cutover, rollback, and post-migration validation.

**Fresh example:** A retailer plans to move a reporting system and a payment system. It rehosts the reporting tier first to reduce datacenter footprint, but redesigns the payment data path separately because the required recovery and security objectives cannot be met by copying its current servers unchanged.

For database migration, compare source and target engine compatibility, schema conversion needs, ongoing change capture, downtime window, and validation strategy. For Oracle workloads, explicitly test required features and licensing assumptions on the proposed target before committing to a managed database service.

## Virtual desktops and application streaming

- **Amazon WorkSpaces** provides a managed virtual desktop for a user who needs a persistent desktop environment.
- **Amazon AppStream 2.0** streams selected applications to users without requiring a full persistent desktop for every user workflow.

Choose based on whether people need a complete desktop or access to a controlled set of applications, and evaluate user identity, data location, network latency, image maintenance, session persistence, and cost. Do not assume that either service automatically solves data-loss, endpoint-security, or licensing requirements.

Amazon WorkDocs appears in the historical study collection as a collaboration and document-management service. Confirm current AWS service lifecycle, regional availability, and migration guidance before treating it as a deployable option; this guide does not reproduce its original material or recommend it for a new design.

## Speech, language, and document services

AWS managed AI services can be combined as stages in a workflow:

- **Amazon Transcribe** converts supported audio into text.
- **Amazon Polly** generates speech from text.
- **Amazon Comprehend** analyzes supported text for language-related signals.
- **Amazon Lex** builds conversational interfaces.
- **Amazon Textract** extracts text and structured information from supported documents.

Model the data flow, consent, retention, encryption, access controls, and human review requirements. A service output should not be treated as guaranteed correct; validate it against the application's risk tolerance.

**Fresh example:** A museum offers an audio guide. Transcribe can produce draft captions from recordings, Comprehend can help categorize feedback, and Polly can generate spoken versions of approved text. Each stage has its own data permissions and quality checks.

## A compact study loop

Build original practice cases from general business needs rather than reusing published questions. For each case, identify the primary constraint, compare two plausible architectures, state the trade-off, and name one fact that would change the recommendation. Verify service limits and exam objectives against AWS's current documentation and exam guide.

## References

- [AWS Application Discovery Service](https://docs.aws.amazon.com/application-discovery/latest/userguide/what-is-appdiscovery.html)
- [AWS Migration Hub](https://docs.aws.amazon.com/migrationhub/latest/ug/whatishub.html)
- [AWS Application Migration Service](https://docs.aws.amazon.com/mgn/latest/ug/what-is-application-migration-service.html)
- [AWS Database Migration Service](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
- [Amazon WorkSpaces](https://docs.aws.amazon.com/workspaces/latest/adminguide/amazon-workspaces.html)
- [Amazon AppStream 2.0](https://docs.aws.amazon.com/appstream2/latest/developerguide/what-is-appstream.html)
- [Amazon Lex](https://docs.aws.amazon.com/lexv2/latest/dg/what-is.html)
- [Amazon Polly](https://docs.aws.amazon.com/polly/latest/dg/what-is.html)
- [Amazon Transcribe](https://docs.aws.amazon.com/transcribe/latest/dg/what-is.html)
- [Amazon Comprehend](https://docs.aws.amazon.com/comprehend/latest/dg/what-is.html)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
