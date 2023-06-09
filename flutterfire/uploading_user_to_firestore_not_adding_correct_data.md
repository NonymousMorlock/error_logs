## PROBLEM

I had a sign up issue where when I created a user, the actual `FirebaseAuth.instance.currentUser` would get updated but 
the data being uploaded to the firestore wasn't being added correctly. Some fields were missing.

```dart
 try {
      final userCred = await _authClient.createUserWithEmailAndPassword(
        email: email,
        password: password,
      );

      await userCred.user?.updateDisplayName(fullName);
      await userCred.user?.updatePhotoURL(kDefaultImage);
      await _setUserData(userCred.user!, email);
```
AND `SET USER DATA` METHOD
```dart
  Future<void> _setUserData(User user, String fallbackEmail) async {
    await _cloudStoreClient.collection('users').doc(user.uid).set(
          LocalUserModel(
            uid: user.uid,
            email: user.email ?? fallbackEmail,
            fullName: user.displayName ?? '',
            profilePic: user.photoURL ?? '',
            points: 0,
          ).toMap(),
        );
  }
```

## SOLUTION

I had to use the `userCred.user?.reload()` method to update the user data before uploading it to firestore.

```dart
 try {
      final userCred = await _authClient.createUserWithEmailAndPassword(
        email: email,
        password: password,
      );

      await userCred.user?.updateDisplayName(fullName);
      await userCred.user?.updatePhotoURL(kDefaultImage);
      await userCred.user?.reload();
      await _setUserData(userCred.user!, email);
```

OR

Just use the `FirebaseAuth.instance.currentUser`

```dart
  Future<void> _setUserData(User user, String fallbackEmail) async {
    await _cloudStoreClient.collection('users').doc(user.uid).set(
          LocalUserModel(
            uid: user.uid,
            email: user.email ?? fallbackEmail,
            fullName: user.displayName ?? '',
            profilePic: user.photoURL ?? '',
            points: 0,
          ).toMap(),
        );
  }
```