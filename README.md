# Firebaes Analytics Tracking Extension

```swift

//
//  Tracking.swift
//  MiracleNight
//
//  Created by paige on 2022/04/14.
//

import SwiftUI
import FirebaseAnalytics

protocol Tracking {
    var tag: String { get }
    var enteredAt: TimeInterval { get }
    func eventOccurred(event: String)
    func eventOccurred(event: String, parameters: [String: Any])
    func setUser(userId: String)
    func setUserProperty(key: String, value: String)
    
    func sendTrackingResult()
}

// MARK: - USER PROPERTY
extension Tracking {
    
    func setUser(userId: String) {
        Analytics.setUserID(userId)
    }
    
    func setUserProperty(key: String, value: String) {
        Analytics.setUserProperty(value, forName: key)
    }
    
}

// MARK: - SCREEN TRACKING
/// 유저가 무슨 화면에 오래있는지 추적..
/// 화면마다의 퍼포먼스 추적...
extension Tracking {
    
    private func leftAt() -> TimeInterval {
        return Date().timeIntervalSince1970
    }
    
    private func stayedFor() -> TimeInterval {
        return Date().timeIntervalSince1970 - enteredAt
    }
    
    func sendTrackingResult() {
        var parameters: [String: Any] = [
            "enteredAt": enteredAt,
            "enteredDate": formatDate(timeInterval: enteredAt),
            "leftAt": leftAt(),
            "leftDate": formatDate(timeInterval: leftAt()),
            "stayedFor": stayedFor()
        ]
        let defaultParameters: [String: Any] = buildParameters(tag: tag)
        
        for (key, value) in defaultParameters {
            parameters[key] = value
        }
        
        Analytics.logEvent(tag, parameters: parameters)
    }
    
}

// MARK: - BUTTON TRACKING EVENT
/// 유저 이벤트 추적..
extension Tracking {
    
    func eventOccurred(event: String) {
        var parameters: [String: Any] = [
            "event": event
        ]
        let defaultParameters: [String: Any] = buildParameters(tag: tag)
        for (key, value) in defaultParameters {
            parameters[key] = value
        }
        Analytics.logEvent(event, parameters: parameters)
    }
    
    func eventOccurred(event: String, parameters: [String: Any]) {
        let additionalParameters: [String: Any] = parameters
        var newParameters: [String: Any] = [ // avoid name collision
            "event": event
        ]
        let defaultParameters: [String: Any] = buildParameters(tag: tag)
        for (key, value) in additionalParameters {
            newParameters[key] = value
        }
        for (key, value) in defaultParameters {
            newParameters[key] = value
        }
        Analytics.logEvent(event, parameters: newParameters)
    }
    
}

// MARK: - TRACKING WHERE USER ENTERED AND EXCITED...
/// 유저 이탈율 추적..
extension Tracking {
    
    func willResignActiveNotification(screenTag: String) {
        Analytics.logEvent("willResignActiveNotification", parameters: buildParameters(tag: screenTag))
    }
    
    func willTerminateNotification(screenTag: String) {
        Analytics.logEvent("willTerminateNotification", parameters: buildParameters(tag: screenTag))
    }
    
    func willEnterForegroundNotification(screenTag: String) {
        Analytics.logEvent("willEnterForegroundNotification", parameters: buildParameters(tag: screenTag))
    }
    
    func didBecomeActiveNotification(screenTag: String) {
        Analytics.logEvent("didBecomeActiveNotification", parameters: buildParameters(tag: screenTag))
    }
    
}


// MARK: - UTIL
extension Tracking {
    
    private func buildParameters(tag: String) -> [String: Any] {
        let systemName: String = UIDevice.current.systemName
        let osVersion: String = UIDevice.current.systemVersion
        let deviceName: String = UIDevice.current.name
        return [
              "tag": tag,
              "language_code": Locale.current.languageCode ?? "none",
              "identifier": Locale.current.identifier,
              "currency_code": Locale.current.currencyCode ?? "none",
              "region": Locale.current.regionCode ?? "none",
              "description": Locale.current.description,
              "device_uuid": UIDevice.current.identifierForVendor?.uuidString ?? "none",
              "system_name": systemName,
              "os_version": osVersion,
              "device_name": deviceName]
    }
    
    private func formatDate(timeInterval: TimeInterval) -> String {
        let dateFormatter: DateFormatter = DateFormatter()
        dateFormatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
        dateFormatter.locale = Locale.current
        let date: Date = Date(timeIntervalSince1970: timeInterval)
        return dateFormatter.string(from: date)
    }
    
    
    
}

extension View where Self: Tracking {
    
    func tracking() -> some View {
        self
            .onReceive(NotificationCenter.default.publisher(for: UIApplication.willResignActiveNotification, object: nil)) { _ in
                willResignActiveNotification(screenTag: tag)
            }
            .onReceive(NotificationCenter.default.publisher(for: UIApplication.willTerminateNotification, object: nil)) { _ in
                willTerminateNotification(screenTag: tag)
            }
            .onReceive(NotificationCenter.default.publisher(for: UIApplication.willEnterForegroundNotification, object: nil)) { _ in
                willEnterForegroundNotification(screenTag: tag)
            }
            .onReceive(NotificationCenter.default.publisher(for: UIApplication.didBecomeActiveNotification, object: nil)) { _ in
                didBecomeActiveNotification(screenTag: tag)
            }
            .onDisappear {
                sendTrackingResult()
            }
    }
    
}


```

### Usage 

```swift

struct MyView: View, Tracking {

    private(set) var tag: String = "MyView"
    private(set) var enteredAt: TimeInterval = Date().timeIntervalSince1970

    var body: some View {
        ZStack {
        
            Button(action: {
                eventOcurred(event: "\(tag)_button_click_event")
            }, label: { 
                Text("Button")
            })
        
        }
        .tracking()
    }
    
}

```
