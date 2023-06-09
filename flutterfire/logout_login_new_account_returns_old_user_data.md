## PROBLEM

I had an app that used firebase, and I has a userDataStream defined in a DashboardUtil,
however, when I logged out and logged in with a new account, the userDataStream would still
return the same data as the old user. I logged every process from sign in to sign out, and
found out that the right user was being pulled from firebase, but the userDataStream was still
returning the old user, it was using the `FirebaseAuth.instance.currentUser!.uid` to get the
user data, and I found out that the `currentUser` was being updated everywhere except there,
in the `DashBoardUtil.userDataStream`

```dart
class DashboardUtils {
  static final userDataStream = FirebaseFirestore.instance
      .collection('users')
      .doc(FirebaseAuth.instance.currentUser!.uid)
      .snapshots()
      .map((event) => LocalUserModel.fromMap(event.data()!));
}
```

## SOLUTION

Apparently, this being a static variable, it was only being initialized once, and it was
not being updated when the `currentUser` changed, so I had to make it a getter, and it worked

```dart
class DashboardUtils {
  static Stream<LocalUserModel> get userDataStream => FirebaseFirestore.instance
      .collection('users')
      .doc(FirebaseAuth.instance.currentUser!.uid)
      .snapshots()
      .map((event) => LocalUserModel.fromMap(event.data()!));
}
```

OR

```dart
class DashboardUtils {
  static Stream<LocalUserModel> userDataStream() => FirebaseFirestore
      .instance
      .collection('users')
      .doc(FirebaseAuth.instance.currentUser!.uid)
      .snapshots()
      .map((event) => LocalUserModel.fromMap(event.data()!));
}
```
