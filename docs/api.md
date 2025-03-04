<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
# API Reference

- [Basic functions](#basic-functions)
  - [javadoc](#javadoc)
  - [java_export](#java_export)
  - [maven_bom](#maven_bom)
  - [maven_install](#maven_install)
- [Maven specification functions](#maven-specification-functions)
  - [maven.repository](#mavenrepository)
  - [maven.artifact](#mavenartifact)
  - [maven.exclusion](#mavenexclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Basic functions

These are the basic functions to get started.

To use these functions, load them at the top of your BUILD file. For example:

```python
load("@rules_jvm_external//:defs.bzl", "maven_install", "artifact")
```
<!-- Generated with Stardoc: http://skydoc.bazel.build -->



<a id="javadoc"></a>

## javadoc

<pre>
javadoc(<a href="#javadoc-name">name</a>, <a href="#javadoc-deps">deps</a>, <a href="#javadoc-javadocopts">javadocopts</a>)
</pre>

Generate a javadoc from all the `deps`

**ATTRIBUTES**


| Name  | Description | Type | Mandatory | Default |
| :------------- | :------------- | :------------- | :------------- | :------------- |
| <a id="javadoc-name"></a>name |  A unique name for this target.   | <a href="https://bazel.build/concepts/labels#target-names">Name</a> | required |  |
| <a id="javadoc-deps"></a>deps |  The java libraries to generate javadocs for.<br><br>          The source jars of each dep will be used to generate the javadocs.           Currently docs for transitive dependencies are not generated.   | <a href="https://bazel.build/concepts/labels">List of labels</a> | required |  |
| <a id="javadoc-javadocopts"></a>javadocopts |  javadoc options.             Note sources and classpath are derived from the deps. Any additional             options can be passed here.   | List of strings | optional | <code>[]</code> |


<a id="java_export"></a>

## java_export

<pre>
java_export(<a href="#java_export-name">name</a>, <a href="#java_export-maven_coordinates">maven_coordinates</a>, <a href="#java_export-deploy_env">deploy_env</a>, <a href="#java_export-pom_template">pom_template</a>, <a href="#java_export-visibility">visibility</a>, <a href="#java_export-tags">tags</a>, <a href="#java_export-testonly">testonly</a>, <a href="#java_export-kwargs">kwargs</a>)
</pre>

Extends `java_library` to allow maven artifacts to be uploaded.

This macro can be used as a drop-in replacement for `java_library`, but
also generates an implicit `name.publish` target that can be run to publish
maven artifacts derived from this macro to a maven repository. The publish
rule understands the following variables (declared using `--define` when
using `bazel run`):

  * `maven_repo`: A URL for the repo to use. May be "https" or "file".
  * `maven_user`: The user name to use when uploading to the maven repository.
  * `maven_password`: The password to use when uploading to the maven repository.

This macro also generates a `name-pom` target that creates the `pom.xml` file
associated with the artifacts. The template used is derived from the (optional)
`pom_template` argument, and the following substitutions are performed on
the template file:

  * `{groupId}`: Replaced with the maven coordinates group ID.
  * `{artifactId}`: Replaced with the maven coordinates artifact ID.
  * `{version}`: Replaced by the maven coordinates version.
  * `{type}`: Replaced by the maven coordinates type, if present (defaults to "jar")
  * `{scope}`: Replaced by the maven coordinates type, if present (defaults to "compile")
  * `{dependencies}`: Replaced by a list of maven dependencies directly relied upon
    by java_library targets within the artifact.

The "edges" of the artifact are found by scanning targets that contribute to
runtime dependencies for the following tags:

  * `maven_coordinates=group:artifact:type:version`: Specifies a dependency of
    this artifact.
  * `maven:compile_only`: Specifies that this dependency should not be listed
    as a dependency of the artifact being generated.

To skip generation of the javadoc jar, add the `no-javadocs` tag to the target.

Generated rules:
  * `name`: A `java_library` that other rules can depend upon.
  * `name-docs`: A javadoc jar file.
  * `name-pom`: The pom.xml file.
  * `name.publish`: To be executed by `bazel run` to publish to a maven repo.


**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="java_export-name"></a>name |  A unique name for this target   |  none |
| <a id="java_export-maven_coordinates"></a>maven_coordinates |  The maven coordinates for this target.   |  none |
| <a id="java_export-deploy_env"></a>deploy_env |  A list of labels of Java targets to exclude from the generated jar. [<code>java_binary</code>](https://bazel.build/reference/be/java#java_binary) targets are *not* supported.   |  <code>[]</code> |
| <a id="java_export-pom_template"></a>pom_template |  The template to be used for the pom.xml file.   |  <code>None</code> |
| <a id="java_export-visibility"></a>visibility |  The visibility of the target   |  <code>None</code> |
| <a id="java_export-tags"></a>tags |  <p align="center"> - </p>   |  <code>[]</code> |
| <a id="java_export-testonly"></a>testonly |  <p align="center"> - </p>   |  <code>None</code> |
| <a id="java_export-kwargs"></a>kwargs |  <p align="center"> - </p>   |  none |


<a id="maven_bom"></a>

## maven_bom

<pre>
maven_bom(<a href="#maven_bom-name">name</a>, <a href="#maven_bom-maven_coordinates">maven_coordinates</a>, <a href="#maven_bom-java_exports">java_exports</a>, <a href="#maven_bom-bom_pom_template">bom_pom_template</a>, <a href="#maven_bom-dependencies_maven_coordinates">dependencies_maven_coordinates</a>,
          <a href="#maven_bom-dependencies_pom_template">dependencies_pom_template</a>, <a href="#maven_bom-tags">tags</a>, <a href="#maven_bom-testonly">testonly</a>, <a href="#maven_bom-visibility">visibility</a>)
</pre>

Generates a Maven BOM `pom.xml` file and an optional "dependencies" `pom.xml`.

The generated BOM will contain a list of all the coordinates of the
`java_export` targets in the `java_exports` parameters. An optional
dependencies artifact will be created if the parameter
`dependencies_maven_coordinates` is set.

Both the BOM and dependencies artifact can be templatised to support
customisation, but a sensible default template will be used if none is
provided. The template used is derived from the (optional)
`pom_template` argument, and the following substitutions are performed on
the template file:

  * `{groupId}`: Replaced with the maven coordinates group ID.
  * `{artifactId}`: Replaced with the maven coordinates artifact ID.
  * `{version}`: Replaced by the maven coordinates version.
  * `{dependencies}`: Replaced by a list of maven dependencies directly relied upon
    by java_library targets within the artifact.

To publish, call the implicit `*.publish` target(s).

The maven repository may accessed locally using a `file://` URL, or
remotely using an `https://` URL. The following flags may be set
using `--define`:

  * `gpg_sign`: Whether to sign artifacts using GPG
  * `maven_repo`: A URL for the repo to use. May be "https" or "file".
  * `maven_user`: The user name to use when uploading to the maven repository.
  * `maven_password`: The password to use when uploading to the maven repository.

When signing with GPG, the current default key is used.

Generated rules:
  * `name`: The BOM file itself.
  * `name.publish`: To be executed by `bazel run` to publish the BOM to a maven repo
  * `name-dependencies`: The BOM file for the dependencies `pom.xml`. Only generated if `dependencies_maven_coordinates` is set.
  * `name-dependencies.publish`: To be executed by `bazel run` to publish the dependencies `pom.xml` to a maven rpo. Only generated if `dependencies_maven_coordinates` is set.


**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="maven_bom-name"></a>name |  A unique name for this rule.   |  none |
| <a id="maven_bom-maven_coordinates"></a>maven_coordinates |  The maven coordinates of this BOM in <code>groupId:artifactId:version</code> form.   |  none |
| <a id="maven_bom-java_exports"></a>java_exports |  A list of <code>java_export</code> targets that are used to generate the BOM.   |  none |
| <a id="maven_bom-bom_pom_template"></a>bom_pom_template |  A template used for generating the <code>pom.xml</code> of the BOM at <code>maven_coordinates</code> (optional)   |  <code>None</code> |
| <a id="maven_bom-dependencies_maven_coordinates"></a>dependencies_maven_coordinates |  The maven coordinates of a dependencies artifact to generate in GAV format. If empty, none will be generated. (optional)   |  <code>None</code> |
| <a id="maven_bom-dependencies_pom_template"></a>dependencies_pom_template |  A template used for generating the <code>pom.xml</code> of the dependencies artifact at <code>dependencies_maven_coordinates</code> (optional)   |  <code>None</code> |
| <a id="maven_bom-tags"></a>tags |  <p align="center"> - </p>   |  <code>None</code> |
| <a id="maven_bom-testonly"></a>testonly |  <p align="center"> - </p>   |  <code>None</code> |
| <a id="maven_bom-visibility"></a>visibility |  <p align="center"> - </p>   |  <code>None</code> |


<a id="maven_install"></a>

## maven_install

<pre>
maven_install(<a href="#maven_install-name">name</a>, <a href="#maven_install-repositories">repositories</a>, <a href="#maven_install-artifacts">artifacts</a>, <a href="#maven_install-fail_on_missing_checksum">fail_on_missing_checksum</a>, <a href="#maven_install-fetch_sources">fetch_sources</a>, <a href="#maven_install-fetch_javadoc">fetch_javadoc</a>,
              <a href="#maven_install-use_unsafe_shared_cache">use_unsafe_shared_cache</a>, <a href="#maven_install-excluded_artifacts">excluded_artifacts</a>, <a href="#maven_install-generate_compat_repositories">generate_compat_repositories</a>,
              <a href="#maven_install-version_conflict_policy">version_conflict_policy</a>, <a href="#maven_install-maven_install_json">maven_install_json</a>, <a href="#maven_install-override_targets">override_targets</a>, <a href="#maven_install-strict_visibility">strict_visibility</a>,
              <a href="#maven_install-strict_visibility_value">strict_visibility_value</a>, <a href="#maven_install-resolve_timeout">resolve_timeout</a>, <a href="#maven_install-jetify">jetify</a>, <a href="#maven_install-jetify_include_list">jetify_include_list</a>,
              <a href="#maven_install-additional_netrc_lines">additional_netrc_lines</a>, <a href="#maven_install-use_credentials_from_home_netrc_file">use_credentials_from_home_netrc_file</a>, <a href="#maven_install-fail_if_repin_required">fail_if_repin_required</a>,
              <a href="#maven_install-use_starlark_android_rules">use_starlark_android_rules</a>, <a href="#maven_install-aar_import_bzl_label">aar_import_bzl_label</a>, <a href="#maven_install-duplicate_version_warning">duplicate_version_warning</a>)
</pre>

Resolves and fetches artifacts transitively from Maven repositories.

This macro runs a repository rule that invokes the Coursier CLI to resolve
and fetch Maven artifacts transitively.


**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="maven_install-name"></a>name |  A unique name for this Bazel external repository.   |  <code>"maven"</code> |
| <a id="maven_install-repositories"></a>repositories |  A list of Maven repository URLs, specified in lookup order.<br><br>Supports URLs with HTTP Basic Authentication, e.g. "https://username:password@example.com".   |  <code>[]</code> |
| <a id="maven_install-artifacts"></a>artifacts |  A list of Maven artifact coordinates in the form of <code>group:artifact:version</code>.   |  <code>[]</code> |
| <a id="maven_install-fail_on_missing_checksum"></a>fail_on_missing_checksum |  fail the fetch if checksum attributes are not present.   |  <code>True</code> |
| <a id="maven_install-fetch_sources"></a>fetch_sources |  Additionally fetch source JARs.   |  <code>False</code> |
| <a id="maven_install-fetch_javadoc"></a>fetch_javadoc |  Additionally fetch javadoc JARs.   |  <code>False</code> |
| <a id="maven_install-use_unsafe_shared_cache"></a>use_unsafe_shared_cache |  Download artifacts into a persistent shared cache on disk. Unsafe as Bazel is currently unable to detect modifications to the cache.   |  <code>False</code> |
| <a id="maven_install-excluded_artifacts"></a>excluded_artifacts |  A list of Maven artifact coordinates in the form of <code>group:artifact</code> to be excluded from the transitive dependencies.   |  <code>[]</code> |
| <a id="maven_install-generate_compat_repositories"></a>generate_compat_repositories |  Additionally generate repository aliases in a .bzl file for all JAR artifacts. For example, <code>@maven//:com_google_guava_guava</code> can also be referenced as <code>@com_google_guava_guava//jar</code>.   |  <code>False</code> |
| <a id="maven_install-version_conflict_policy"></a>version_conflict_policy |  Policy for user-defined vs. transitive dependency version conflicts.  If "pinned", choose the user's version unconditionally.  If "default", follow Coursier's default policy.   |  <code>"default"</code> |
| <a id="maven_install-maven_install_json"></a>maven_install_json |  A label to a <code>maven_install.json</code> file to use pinned artifacts for generating build targets. e.g <code>//:maven_install.json</code>.   |  <code>None</code> |
| <a id="maven_install-override_targets"></a>override_targets |  A mapping of <code>group:artifact</code> to Bazel target labels. All occurrences of the target label for <code>group:artifact</code> will be an alias to the specified label, therefore overriding the original generated <code>jvm_import</code> or <code>aar_import</code> target.   |  <code>{}</code> |
| <a id="maven_install-strict_visibility"></a>strict_visibility |  Controls visibility of transitive dependencies. If <code>True</code>, transitive dependencies are private and invisible to user's rules. If <code>False</code>, transitive dependencies are public and visible to user's rules.   |  <code>False</code> |
| <a id="maven_install-strict_visibility_value"></a>strict_visibility_value |  Allows changing transitive dependencies strict visibility scope from private to specified scopes list.   |  <code>["//visibility:private"]</code> |
| <a id="maven_install-resolve_timeout"></a>resolve_timeout |  The execution timeout of resolving and fetching artifacts.   |  <code>600</code> |
| <a id="maven_install-jetify"></a>jetify |  Runs the AndroidX [Jetifier](https://developer.android.com/studio/command-line/jetifier) tool on artifacts specified in jetify_include_list. If jetify_include_list is not specified, run Jetifier on all artifacts.   |  <code>False</code> |
| <a id="maven_install-jetify_include_list"></a>jetify_include_list |  List of artifacts that need to be jetified in <code>groupId:artifactId</code> format. By default all artifacts are jetified if <code>jetify</code> is set to True.   |  <code>["*"]</code> |
| <a id="maven_install-additional_netrc_lines"></a>additional_netrc_lines |  Additional lines prepended to the netrc file used by <code>http_file</code> (with <code>maven_install_json</code> only).   |  <code>[]</code> |
| <a id="maven_install-use_credentials_from_home_netrc_file"></a>use_credentials_from_home_netrc_file |  Whether to pass machine login credentials from the ~/.netrc file to coursier.   |  <code>False</code> |
| <a id="maven_install-fail_if_repin_required"></a>fail_if_repin_required |  Whether to fail the build if the required maven artifacts have been changed but not repinned. Requires the <code>maven_install_json</code> to have been set.   |  <code>False</code> |
| <a id="maven_install-use_starlark_android_rules"></a>use_starlark_android_rules |  Whether to use the native or Starlark version of the Android rules. Default is False.   |  <code>False</code> |
| <a id="maven_install-aar_import_bzl_label"></a>aar_import_bzl_label |  The label (as a string) to use to import aar_import from. This is usually needed only if the top-level workspace file does not use the typical default repository name to import the Android Starlark rules. Default is "@build_bazel_rules_android//rules:rules.bzl".   |  <code>"@build_bazel_rules_android//android:rules.bzl"</code> |
| <a id="maven_install-duplicate_version_warning"></a>duplicate_version_warning |  What to do if an artifact is specified multiple times. If "error" then fail the build, if "warn" then print a message and continue, if "none" then do nothing. The default is "warn".   |  <code>"warn"</code> |


# Maven specification functions

These are helper functions to specify more information about Maven artifacts and
repositories in `maven_install`.

To use these functions, load the `maven` struct at the top of your BUILD file:

```python
load("@rules_jvm_external//:specs.bzl", "maven")
```
<!-- Generated with Stardoc: http://skydoc.bazel.build -->



<a id="maven.repository"></a>

## maven.repository

<pre>
maven.repository(<a href="#maven.repository-url">url</a>, <a href="#maven.repository-user">user</a>, <a href="#maven.repository-password">password</a>)
</pre>

Generates the data map for a Maven repository specifier given the available information.

If both a user and password are given as arguments, it will include the
access credentials in the repository spec. If one or both are missing, it
will just generate the repository url.


**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="maven.repository-url"></a>url |  A string containing the repository url (ex: "https://maven.google.com/").   |  none |
| <a id="maven.repository-user"></a>user |  A username for this Maven repository, if it requires authentication (ex: "johndoe").   |  <code>None</code> |
| <a id="maven.repository-password"></a>password |  A password for this Maven repository, if it requires authentication (ex: "example-password").   |  <code>None</code> |


<a id="maven.artifact"></a>

## maven.artifact

<pre>
maven.artifact(<a href="#maven.artifact-group">group</a>, <a href="#maven.artifact-artifact">artifact</a>, <a href="#maven.artifact-version">version</a>, <a href="#maven.artifact-packaging">packaging</a>, <a href="#maven.artifact-classifier">classifier</a>, <a href="#maven.artifact-override_license_types">override_license_types</a>, <a href="#maven.artifact-exclusions">exclusions</a>,
               <a href="#maven.artifact-neverlink">neverlink</a>, <a href="#maven.artifact-testonly">testonly</a>)
</pre>

Generates the data map for a Maven artifact given the available information about its coordinates.

**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="maven.artifact-group"></a>group |  The Maven artifact coordinate group name (ex: "com.google.guava").   |  none |
| <a id="maven.artifact-artifact"></a>artifact |  The Maven artifact coordinate artifact name (ex: "guava").   |  none |
| <a id="maven.artifact-version"></a>version |  The Maven artifact coordinate version name (ex: "27.0-jre").   |  none |
| <a id="maven.artifact-packaging"></a>packaging |  The Maven packaging specifier (ex: "jar").   |  <code>None</code> |
| <a id="maven.artifact-classifier"></a>classifier |  The Maven artifact classifier (ex: "javadoc").   |  <code>None</code> |
| <a id="maven.artifact-override_license_types"></a>override_license_types |  An array of Bazel license type strings to use for this artifact's rules (overrides autodetection) (ex: ["notify"]).   |  <code>None</code> |
| <a id="maven.artifact-exclusions"></a>exclusions |  An array of exclusion objects to create exclusion specifiers for this artifact (ex: maven.exclusion("junit", "junit")).   |  <code>None</code> |
| <a id="maven.artifact-neverlink"></a>neverlink |  Determines if this artifact should be part of the runtime classpath.   |  <code>None</code> |
| <a id="maven.artifact-testonly"></a>testonly |  Determines whether this artifact is available for targets not marked as <code>testonly = True</code>.   |  <code>None</code> |


<a id="maven.exclusion"></a>

## maven.exclusion

<pre>
maven.exclusion(<a href="#maven.exclusion-group">group</a>, <a href="#maven.exclusion-artifact">artifact</a>)
</pre>

Generates the data map for a Maven artifact exclusion.

**PARAMETERS**


| Name  | Description | Default Value |
| :------------- | :------------- | :------------- |
| <a id="maven.exclusion-group"></a>group |  The Maven group name of the dependency to exclude, e.g. "com.google.guava".   |  none |
| <a id="maven.exclusion-artifact"></a>artifact |  The Maven artifact name of the dependency to exclude, e.g. "guava".   |  none |


