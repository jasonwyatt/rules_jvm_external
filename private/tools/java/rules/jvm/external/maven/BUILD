load("@rules_jvm_external//:defs.bzl", "artifact")

java_binary(
    name = "MavenPublisher",
    srcs = ["MavenPublisher.java"],
    main_class = "rules.jvm.external.maven.MavenPublisher",
    visibility = ["//visibility:public"],
    deps = [
        "//private/tools/java/rules/jvm/external:byte-streams",
        artifact(
            "com.google.auth:google-auth-library-oauth2-http",
            repository_name = "rules_jvm_external_deps",
        ),
        artifact(
            "com.google.auth:google-auth-library-credentials",
            repository_name = "rules_jvm_external_deps",
        ),
        artifact(
            "com.google.cloud:google-cloud-storage",
            repository_name = "rules_jvm_external_deps",
        ),
        artifact(
            "com.google.cloud:google-cloud-core",
            repository_name = "rules_jvm_external_deps",
        ),
        artifact(
            "software.amazon.awssdk:s3",
            repository_name = "rules_jvm_external_deps",
        ),
    ],
)

java_binary(
    name = "outdated",
    srcs = ["Outdated.java"],
    main_class = "rules.jvm.external.maven.Outdated",
    visibility = ["//visibility:public"],
    deps = [
        artifact(
            "org.apache.maven:maven-artifact",
            repository_name = "rules_jvm_external_deps",
        ),
    ],
)
