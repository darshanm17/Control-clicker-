import Foundation
import Cocoa
import ApplicationServices

func getControlCenterProcess() -> AXUIElement? {
    let apps = NSWorkspace.shared.runningApplications
    for app in apps {
        if app.bundleIdentifier == "com.apple.controlcenter" {
            return AXUIElementCreateApplication(app.processIdentifier)
        }
    }
    return nil
}

func dumpButtons(in element: AXUIElement, depth: Int = 0) {
    var children: CFTypeRef?
    let result = AXUIElementCopyAttributeValue(element, kAXChildrenAttribute as CFString, &children)

    guard result == .success, let childrenArray = children as? [AXUIElement] else {
        return
    }

    for child in childrenArray {
        var role: CFTypeRef?
        AXUIElementCopyAttributeValue(child, kAXRoleAttribute as CFString, &role)

        if let role = role as? String, role == kAXButtonRole as String {
            var title: CFTypeRef?
            if AXUIElementCopyAttributeValue(child, kAXTitleAttribute as CFString, &title) == .success,
               let name = title as? String {
                print("🔘 Button found: \(String(repeating: " ", count: depth * 2))\(name)")
            }
        }

        // Recursively search deeper levels
        dumpButtons(in: child, depth: depth + 1)
    }
}

func findControlCenterWindow() -> AXUIElement? {
    guard let controlCenter = getControlCenterProcess() else {
        print("❌ Control Center app not found.")
        return nil
    }

    var windows: CFTypeRef?
    AXUIElementCopyAttributeValue(controlCenter, kAXWindowsAttribute as CFString, &windows)

    guard let windowArray = windows as? [AXUIElement], let firstWindow = windowArray.first else {
        print("❌ No Control Center windows found.")
        return nil
    }

    return firstWindow
}

// MARK: - MAIN
print("🔍 Looking for Control Center devices...")
if let ccWindow = findControlCenterWindow() {
    dumpButtons(in: ccWindow)
} else {
    print("❌ Couldn't access Control Center window.")
}
