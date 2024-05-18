---
layout: post
comments: true
title: "UIPageViewController: Customizing Page Indicators"
description: customize page indicator in UIPageViewController
summary: customize page indicator in UIPageViewController
tags: [swift, uikit]
---

Having not worked on an iOS project in a long while, I was stumped recently when I had to customize the page indicator for an onboarding screen. 

I tend to build my views programatically but documentation for UIPageViewController does not provide information on how to programatically access the default page indication built into UIPageViewController. My guess, after trying different methods on my instance of UIPageViewController and having my google-fu fail me, is that there isn't a way to programatically access the default page indicator. 

The solution I implemented relies on both `presentationCount` and `presentationIndex`. According to Apple's documentation a page indicator is visible under these conditions only: 
	- if both `presentationCount` and `presentationIndex` are implemented
	- if the transitionStyle is scroll 
	- and if the navigation orientation is horizontal.

As of iOS 16, the minimal code to accomplish this: 

```
import UIKit

class OnboardingViewController: UIViewController {

  private var currentIndex: Int = 0
  private var pageController: UIPageViewController?

  override func viewDidLoad() {
    super.viewDidLoad()
    self.setupPageController()
  }

  private func setupPageController() {
    self.pageController = UIPageViewController(
      transitionStyle: .scroll, navigationOrientation: .horizontal)
    self.pageController?.delegate = self
    self.pageController?.dataSource = self
    self.pageController?.view.backgroundColor = .systemRed
  }

  private func customizePageIndicator() {
    let appearance = UIPageControl.appearance()
    appearance.pageIndicatorTintColor = .systemGray
    appearance.currentPageIndicatorTintColor = .white
    appearance.backgroundColor = .systemRed
  }

  func presentationCount(for pageViewController: UIPageViewController) -> Int {
    self.customizePageIndicator()
    return self.onboardingContent.count
  }

  func presentationIndex(for pageViewController: UIPageViewController) -> Int {
    return self.currentIndex
  }
}

extension OnboardingViewController: UIPageViewControllerDataSource {

  func pageViewController(
    _ pageViewController: UIPageViewController, viewControllerAfter viewController: UIViewController
  ) -> UIViewController? {
    // implement logic that displays next vc when user swipes to next screen
  }

  func pageViewController(
    _ pageViewController: UIPageViewController,
    viewControllerBefore viewController: UIViewController
  ) -> UIViewController? {
    //implement logic that displays previous viewController
    // basically what to display when user swips back to previous screen
  }
}     
```
**note**: I am using the regular ViewController as the container of the PageViewController; you don't have to do it this way.

It feels counter-intutive to implement a new UIPageControl instead of using what is baked into the UIPageViewController; will have to investigate when time permits.