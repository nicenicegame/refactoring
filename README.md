# Refactoring Assignment

## FlashGet
From my FlashGet code: https://github.com/nicenicegame/pa4-nicenicegame

In the `src/flashget/Controller.java` class
https://github.com/nicenicegame/pa4-nicenicegame/blob/master/src/flashget/Controller.java

### download() Method
consider this code:
```java
    public void download(ActionEvent event) {
        // ...
            // if the save field is empty, there will be the confirmation dialog to ask for set path to initial
            if (!urlField.getText().isEmpty() && saveField.getText().isEmpty()) {
                Alert alert = new Alert(Alert.AlertType.CONFIRMATION);
                alert.setTitle("Confirmation Dialog");
                alert.setHeaderText("File path is not set up.");
                alert.setContentText("The path will be set to system properties. Are you OK?");

                Optional<ButtonType> result = alert.showAndWait();
                try {
                    if (result.isPresent() && result.get() == ButtonType.OK) {
                        out = new File(System.getProperty("user.home") + "\\" + filename);
                        saveField.setText(out.getPath());
                    }
                } catch (NullPointerException ignored) {
                }
                // if both field are fill up, it will start downloading
            } else if (!urlField.getText().isEmpty() && !saveField.getText().isEmpty()) {
                // ...
        }
    }
```

- Refactoring sign
    - the download method should do the download work, not show the confirmation dialog.

- Refactoring
    - extract the method and named `showConfirmationDialog` and call this new method instead.
```java
    private void showConfirmationDialog(String filename) {
        Alert alert = new Alert(Alert.AlertType.CONFIRMATION);
        alert.setTitle("Confirmation Dialog");
        alert.setHeaderText("File path is not set up.");
        alert.setContentText("The path will be set to system properties. Are you OK?");

        Optional<ButtonType> result = alert.showAndWait();
        try {
            if (result.isPresent() && result.get() == ButtonType.OK) {
                out = new File(System.getProperty("user.home") + "\\" + filename);
                saveField.setText(out.getPath());
            }
        } catch (NullPointerException ignored) {
        }
    }
``` 

- After refactoring
```java
    public void download(ActionEvent event) {
        // ...
            // if the save field is empty, there will be the confirmation dialog to ask for set path to initial
            if (!urlField.getText().isEmpty() && saveField.getText().isEmpty()) {
                showConfirmationDialog(filename);
                // if both field are fill up, it will start downloading
            } else if (!urlField.getText().isEmpty() && !saveField.getText().isEmpty()) {
                // ...
        }
    }
```

### download() and cancel() Method
consider this code:
```java
    public void download(ActionEvent event) {
        // ...
            } else if (!urlField.getText().isEmpty() && !saveField.getText().isEmpty()) {
                threadLabel.setVisible(true);
                progressBar.setVisible(true);
                cancelButton.setVisible(true);
                filenameLabel.setVisible(true);
                progressLabel.setVisible(true);
                // ...
                for (ProgressBar progressBar : progressBars) {
                    progressBar.setVisible(true);
                }
                // ...
            }
        }
    }
```
and this code:
```java
    public void cancel(ActionEvent event) {
        // ...
        for (ProgressBar progressBar : progressBars) {
            progressBar.setVisible(false);
        }
        threadLabel.setVisible(false);
        filenameLabel.setVisible(false);
        progressLabel.setVisible(false);
        progressBar.setVisible(false);
        cancelButton.setVisible(false);
    }
```

- Refactoring sign
    - the code is duplicated.
    - set visible is not the work of download and cancel method.
    
- Refactoring
    - extract the method and named `toggleVisible` and call this new method instead.
```java
    private void toggleVisible(boolean visibility) {
        for (ProgressBar progressBar : progressBars) {
            progressBar.setVisible(visibility);
        }
        threadLabel.setVisible(visibility);
        filenameLabel.setVisible(visibility);
        progressLabel.setVisible(visibility);
        progressBar.setVisible(visibility);
        cancelButton.setVisible(visibility);
    }
```
    
After refactoring
```java
    public void download(ActionEvent event) {
        // ...
            } else if (!urlField.getText().isEmpty() && !saveField.getText().isEmpty()) {
                toggleVisible(true);
                // ...
            }
        }
    }
```
```java
    public void cancel(ActionEvent event) {
        // ...
        toggleVisible(false);
    }
```

In the `src/flashget/DownloaderFactory.java` class

https://github.com/nicenicegame/pa4-nicenicegame/blob/master/src/flashget/DownloaderFactory.java

### getDownloadTasks() Method
consider this code:
```java
    public DownloadTask[] getDownloadTasks(URL fileUrl, File out, long fileSize, int numThread) {
        DownloadTask[] downloadTasks = new DownloadTask[numThread];
        long chunk = fileSize / numThread;
        for (int i = 0; i < numThread; i++) {
            if (i == numThread - 1) {
                downloadTasks[i] = new DownloadTask(fileUrl, out, (chunk * (numThread - 1)) + 1, fileSize - (chunk * (numThread - 1)));
            } else {
                downloadTasks[i] = new DownloadTask(fileUrl, out, (chunk * i) + 1, chunk);
            }
        }
        return downloadTasks;
    }
```

- Refactoring sign
    - this method calculate chunks and use temporary variable.
    
- Refactoring
    - Move the entire expression to a separate method and return the result from it.
```java
    private long getChunk(long fileSize, int numThread) {
        return fileSize / numThread;
    }
```

After refactoring
```java
    public DownloadTask[] getDownloadTasks(URL fileUrl, File out, long fileSize, int numThread) {
        DownloadTask[] downloadTasks = new DownloadTask[numThread];
        long chunk = getChunk(fileSize, numThread);
        for (int i = 0; i < numThread; i++) {
            if (i == numThread - 1) {
                downloadTasks[i] = new DownloadTask(fileUrl, out, (chunk * (numThread - 1)) + 1, fileSize - (chunk * (numThread - 1)));
            } else {
                downloadTasks[i] = new DownloadTask(fileUrl, out, (chunk * i) + 1, chunk);
            }
        }
        return downloadTasks;
    }
```