# 获取配置文件的路径列表
$configFiles = Get-ChildItem "C:\inetpub\temp\appPools" -Recurse -Include *.config

# 定义模块信息
$modules = @(
    @{
        Name = "ScriptModule-2.0"
        Image = "%windir%\Microsoft.NET\Framework\v2.0.50727\mscorst.dll"
        PreCondition = "bitness32"
    },
    @{
        Name = "IsapiCacheModule"
        Image = "%windir%\Microsoft.NET\Framework64\v2.0.50727\mscorst64.dll"
        PreCondition = "bitness64"
    }
)

foreach ($configFile in $configFiles) {
    # 加载配置文件为 XML 对象
    [xml]$configXml = Get-Content -Path $configFile.FullName

    # 向 <globalModules> 元素中添加 <add> 元素
    foreach ($module in $modules) {
        $moduleName = $module.Name
        $imagePath = $module.Image
        $preCondition = $module.PreCondition

        # 检查是否已经存在相同的模块
        $existingModule = $configXml.SelectSingleNode("//add[@name='$moduleName']")
        if ($existingModule -eq $null) {
            # 如果模块不存在，创建新的模块元素
            $newModule = $configXml.CreateElement("add")
            $newModule.SetAttribute("name", $moduleName)
            $newModule.SetAttribute("image", $imagePath)  # 将 type 更改为 image
            $newModule.SetAttribute("preCondition", $preCondition)

            # 查找 <globalModules> 元素并在其下添加模块信息
            $globalModulesElement = $configXml.SelectSingleNode("//globalModules")
            $globalModulesElement.AppendChild($newModule)

            # 如果需要添加到 <modules> 元素中，也可以添加
             $modulesElement = $configXml.SelectSingleNode("//modules")
             $moduleEntry = $configXml.CreateElement("add")
             $moduleEntry.SetAttribute("name", $moduleName)
             $moduleEntry.SetAttribute("preCondition", $preCondition)
             $modulesElement.AppendChild($moduleEntry)
        }
    }

    # 保存修改后的配置文件
    $configXml.Save($configFile.FullName)
}
