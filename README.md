# javastarter
### Init Java Application
> $ gradle init --type java-library
### Java Support for Vscode
> add gradle tasks(java Support extension doc) to build.gradle
task vscodeClasspathFile {
    description 'Generates classpath file for the Visual Studio Code java plugin'
    ext.destFile = file("$buildDir/classpath.txt")
    outputs.file destFile
    doLast {
        def classpathString = configurations.compile.collect{ it.absolutePath }.join(':')
        assert destFile.parentFile.mkdir()
        destFile.text = classpathString
    }
}

task vscodeJavaconfigFile(dependsOn: vscodeClasspathFile) {
    description 'Generates javaconfig.json file for the Visual Studio Code java plugin'

    def relativePath = { File f ->
        f.absolutePath - "${project.rootDir.absolutePath}/"
    }
    ext.destFile = file("javaconfig.json")
    ext.config = [
        sourcePath: sourceSets.collect{ it.java.srcDirs }.flatten().collect{ relativePath(it) },
        classPathFile: relativePath(tasks.getByPath(':vscodeClasspathFile').outputs.files.singleFile),
        outputDirectory: relativePath(new File(buildDir, 'vscode-classes'))
    ]
    doLast {
        def jsonContent = groovy.json.JsonOutput.toJson(ext.config)
        destFile.text = groovy.json.JsonOutput.prettyPrint(jsonContent)
    }
}

task vscode(dependsOn: vscodeJavaconfigFile) {
    description 'Generates config files for the Visual Studio Code java plugin'
    group 'vscode'
}
> Then run gradlew vscode. This will generate
 javaconfig.json
 build/classpath.txt