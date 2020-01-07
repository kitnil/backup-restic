def exclude = ["/home/oleg/.cache",
               "/home/oleg/Downloads",
               "/home/oleg/Downloads",
               "/home/oleg/GNS3/images",
               "/home/oleg/GNS3/projects",
               "/home/oleg/archive/src",
               "/home/oleg/src/replicant",
               "/home/oleg/vm",
               "/home/oleg/Videos",
               "/home/oleg/.guix-profile",
               "/home/oleg/.nix-profile",
               "/home/oleg/src/guix",
               "/home/oleg/src/guix-wip-forge",
               "/home/oleg/src/guix-wip-origin",
               "/home/oleg/src/guix-wip-channels",
               "/home/oleg/src/macos-simple-kvm"].collect{"--exclude $it"}.join(" ")

pipeline {
    agent {
        label "master"
    }
    environment {
        RESTIC_PASSWORD = credentials("RESTIC_PASSWORD")
        RESTIC = "/home/oleg/.guix-profile/bin/restic"
        REPO = "/srv/backup/guixsd"
        SOURCE = "/home/oleg"
    }
    stages {
        stage("Pulling from upstream Git") {
            steps {
                sh (["sudo", "--login", "--preserve-env=RESTIC_PASSWORD",
                     RESTIC, "--repo", REPO, exclude, "backup", SOURCE].join(" "))
            }
        }
    }
    post {
        always {
            sendNotifications currentBuild.result
        }
    }
}
