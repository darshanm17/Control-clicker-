# Control-clicker-
// main.swift
import Foundation
import ApplicationServices
import Cocoa

func findMenuBarItem(named itemName: String) -> AXUIElement? {
    let systemUIApp = NSRunningApplication.runningApplications(withBundleIdentifier: "com.apple.systemuiserver").first
    guard let pid = systemUIApp?.processIdentifier else {
        print("SystemUIServer not found.")
        return nil
    }

    let appElement = AXUIElementCreateApplication(pid)

    var menuBars: CFTypeRef?
    let result = AXUIElementCopyAttributeValue(appElement, kAXMenuBarAttribute as CFString, &menuBars)

    guard result == .success, let menuBar = menuBars else {
        print("Failed to get menu bar.")
        return nil
    }

    var items: CFTypeRef?
    let childrenResult = AXUIElementCopyAttributeValue(menuBar as! AXUIElement, kAXChildrenAttribute as CFString, &items)

    guard childrenResult == .success, let itemArray = items as? [AXUIElement] else {
        print("Failed to get menu bar items.")
        return nil
    }

    for item in itemArray {
        var title: CFTypeRef?
        let nameResult = AXUIElementCopyAttributeValue(item, kAXDescriptionAttribute as CFString, &title)

        if nameResult == .success,
           let name = title as? String,
           name.lowercased().contains(itemName.lowercased()) {
            return item
        }
    }

    return nil
}

func clickControlCenter() {
    guard let controlCenterItem = findMenuBarItem(named: "control center") else {
        print("Control Center menu item not found.")
        return
    }

    let result = AXUIElementPerformAction(controlCenterItem, kAXPressAction as CFString)
    if result == .success {
        print("✅ Clicked Control Center!")
    } else {
        print("❌ Failed to click Control Center.")
    }
}

// MARK: - Run it
clickControlCenter()
