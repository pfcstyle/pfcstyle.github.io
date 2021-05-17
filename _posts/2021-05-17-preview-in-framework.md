---
layout:		post
title:		"Unable to see XCode/SwiftUI Previews within CocoaPods frameworks"
description: ""
date:		2021-05-17
author:		"Yawei"
categories: ["iOS"]
keywords:
    - iOS
    - SwfitUI
    - Preview
    - CocoaPods
---

https://github.com/CocoaPods/CocoaPods/issues/9275#issuecomment-691032405

Add this code in your Podfile:

```
source 'https://github.com/CocoaPods/Specs.git'

# Add this part ====================================
class Pod::Target::BuildSettings::AggregateTargetSettings
  alias_method :ld_runpath_search_paths_original, :ld_runpath_search_paths

  def ld_runpath_search_paths
      return ld_runpath_search_paths_original unless configuration_name == "Debug"
      return (ld_runpath_search_paths_original || []) + (framework_search_paths || [])
  end
end

class Pod::Target::BuildSettings::PodTargetSettings
  alias_method :ld_runpath_search_paths_original, :ld_runpath_search_paths

  def ld_runpath_search_paths
      return (ld_runpath_search_paths_original || []) + (framework_search_paths || [])
  end
end

# Add this part ===========================

...

platform :ios, '14.1'

# Write CommonPods dependencies here
pod 'ArcGIS-Runtime-SDK-iOS', '100.10'
pod 'Alamofire', '~> 5.2'
...

target 'yourFramework' do
...

end

```