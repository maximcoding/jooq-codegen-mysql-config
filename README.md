# Gradle jooq codegen

```buildscript {
    repositories {
        maven {
            name "Artifactory-plugins"
            credentials {
                username = "${artifactory_user}"
                password = "${artifactory_password}"
            }
            url "$artifactory_contextUrl/gradle-plugin-local/"
            allowInsecureProtocol true
        }
        mavenLocal()
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath 'org.jooq:jooq-codegen:3.16.2'
        classpath 'mysql:mysql-connector-java:8.0.32'
    }
}

plugins {
    id 'java-library'
    id 'com.jfrog.artifactory' version '4.29.4'
    id 'nu.studer.jooq' version '8.1'
}
group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
    maven {
        credentials {
            username = "${artifactory_user}"
            password = "${artifactory_password}"
        }
        url "${artifactory_contextUrl}/gradle-dev-local"
        allowInsecureProtocol true
    }
}

dependencies {
    implementation 'org.locationtech.proj4j:proj4j:1.2.2'
    implementation group: 'org.apache.commons', name: 'commons-lang3', version: '3.0'
    implementation 'org.apache.httpcomponents:httpclient:4.5.14'
    implementation 'org.apache.commons:commons-lang3:3.12.0'
    implementation group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.0.1'
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.8.1'
    testImplementation 'junit:junit:4.13.1'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.8.1'
    // JOOQ & MySQL
    implementation group: 'mysql', name: 'mysql-connector-java', version: '8.0.32'
    implementation group: 'org.jooq', name: 'jooq', version: '3.16.2'
    implementation group: 'org.jooq', name: 'jooq-meta', version: '3.16.2'
    compileOnly group: 'org.jooq', name: 'jooq-codegen', version: '3.16.2'
    jooqGenerator("mysql:mysql-connector-java:8.0.32")
}

test {
    useJUnitPlatform()
}

jooq {
    def props = new Properties()
    file("gradle.properties").withInputStream {
        props.load(it)
    }
    version = jooqGeneratorVersion // the default
    edition = nu.studer.gradle.jooq.JooqEdition.OSS  // the default
    configurations {
        main {  // name of the jOOQ configuration
            generateSchemaSourceOnCompilation = true  // default
            generationTool {
                jdbc {
                    driver = 'com.mysql.cj.jdbc.Driver'
                    url = props['databaseUrl']
                    user = props['databaseUser']
                    password = props['databasePass']
                }
                generator {
                    name = 'org.jooq.codegen.DefaultGenerator'
                    database {
                        name = 'org.jooq.meta.mysql.MySQLDatabase'
                        schemata {
                            def schemas = props['databaseSchemas'].split(',')
                            for (item in schemas) {
                                schema {
                                    inputSchema = item
                                }
                            }
                        }
                    }
                    generate {
                        deprecated = false
                        records = true
                    }
                    target {
                        packageName = 'nu.studer.jooq-generated'
                        directory = props['jooqGenerateDir']  // default
                    }
                    strategy.name = 'org.jooq.codegen.DefaultGeneratorStrategy'
                }
            }
        }
    }
}``
