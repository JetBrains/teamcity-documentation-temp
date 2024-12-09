//]: # (title: Visual C Build Issues)
<img width="708" alt="17337039849578010327460780303300" src="https://github.com/user-attachments/assets/8d15f58b-d081-4dd6-94ac-0dc0fd6190b6">
/[/]: # (auxiliary-id: Visual C Build Issues)

If you experience any problems running a Visual C++ build on a build agent, you can try to work around these issues with the following steps, sequentially:

<note>

If none of the steps is helpful for your case, please contact us via any [feedback channel](feedback.md).
</note>
	
* Make sure you are not using mapped network drives.
* Make sure the build user have enough rights to access necessary network paths.
* Sign in to the build agent machine under the same user as that of the build and try running the following command:

    ```Shell
    
    msbuild.exe <path to solution.sln> /p:Configuration:Release /t:Rebuild
    
    ```

* Ensure that the Build Agent service runs under the user with local administrative privileges.
* Check that Microsoft Visual Studio is installed on the build agent.
* Note that you have to start Visual Studio 2005 or Visual Studio 2008 under the build user [at least once](https://devnet.jetbrains.net/message/5233781#5233781).
* If __"Error spawning cmd.exe"__ appears, put the [following lines](https://devnet.jetbrains.net/message/5217957#5217957) exactly into the list in __Tools | Options | Projects and Solutions | VC++ Directories__:

    ```Shell
    --$(SystemRoot)\System32
    --$(SystemRoot)
    --$(SystemRoot)\System32\wbem

    ```
	
* Add all environment variables from `...\Microsoft Visual Studio 9.0\VC\vcvarsall.bat` to the environment or to the [Build Agent Configuration](configure-agent-installation.md).
* Try using the `devenv` command of the with [.NET](net.md) build runner instead of the [Visual Studio (sln)](visual-studio-sln.md) runner.
* Ensure all paths to sources do not contain spaces.
* Set `VCBuildUserEnvironment=true` in the runner properties.
* Specify the `VCBuildAdditionalOptions` property with the value `/useenv` in the build configuration settings to instruct MSBuild to add the `/useenv` command-line argument for spawned VCBuild processes.

 __See also:__

__Administrator's Guide__: [.NET Testing Frameworks Support](net-testing-frameworks-support.md) | [NUnit support](nunit-support.md)
