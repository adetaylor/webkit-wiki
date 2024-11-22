(old proposal for quirks metadata generation I had lying around)

To make maintenance and auditing more straightforward, this proposes a scheme for generating Quirks metadata currently hard coded in WebCore.

Goals:
- Declarative definitions for quirks and adoption.
- Compiles adoptions at build time into equivalent of what we have today, including all caching.
- Optionally supports runtime set of quirk adoptions, allowing for users and/or extensions to enable existing quirks for other sites. 

<hr>

1. Quirk Definitions

The first input for generation will be a quirk definitions file. The file will be called QuirkDefinitions.yaml and contain fields for defining the quirk:

```yaml
- QuirkName
    - description: "Human readable description of what enabling this quirk does"
    - originating-issue: http://webkit.org/b/bug-number
```

2. Quirk Adoption

The second input for generation will a quirk adoption file. The file will be called QuirkAdoption.yaml and contains field for opting into the quirks.

```yaml
- QuirkName # name from the definitions file.
    - AdoptionName1
        - issue: http://webkit.org/b/bug-number
        - platforms: [ list of platforms adoption is limited to ] # Can be excluded if all platforms require adoption
        - predicate:
            - or / and: # nested predicates below these
            - is-top-document: true/false
            - [top-document-]?[host|domain|path|fragment]-[is|contains|starts-with|ends-with]: ...
            - [top-document-]?allowed-autoplay-quirks-contains: [SynthesizedPauseEvents|InheritedUserGestures|...]
    - AdoptionName2
        - issue: http://webkit.org/b/another-bug-number
        - platforms: [ ... ]
        - predicate: [ ... ]
```

<hr>

Examples:

##### QuirkDefinitions.yaml

```yaml
- NeedsFormControlToBeMouseFocusable
    - description: "Quirk for sites that need all form controls to be focusable by a mouse"
    - originating-issue: https://webkit.org/b/????

- NeedsPerDocumentAutoplayBehavior
    - description: "Quirk for sites that need autoplay to be per-document"
    - originating-issue: https://webkit.org/b/????
```

##### QuirkAdoption.yaml

```yaml
- NeedsFormControlToBeMouseFocusable
    - "ceac.state.gov"
       - issue: http://webkit.org/b/193478
       - platforms: [ MAC ]
       - predicate:
            - or:
                - top-document-host-is: "ceac.state.gov"
                - top-document-host-ends-with: ".ceac.state.gov"
    - "weather.com"
       - issue: http://webkit.org/b/?????
       - platforms: [ MAC ]
       - predicate:
            - top-document-host-is: "weather.com"

- NeedsPerDocumentAutoplayBehavior
    - "mac"
       - issue: http://webkit.org/b/193301
       - platforms: [ MAC ]
       - predicate:
           - top-document-allowed-autoplay-quirks-contains: "PerDocumentAutoplayBehavior"
    - "Netflix"
       - issue: http://webkit.org/b/???
       - platforms: [ !MAC ]
       - predicate:
           - domain-is: "netflix.com"
```
