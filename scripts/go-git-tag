#!/usr/bin/env -S pkgx +go@1.22 scriptisto

package go_git_tag

import (
	"errors"
	"fmt"
	"github.com/manifoldco/promptui"
	"log"
	"os"
	"os/exec"
	"strings"
)

func main() {
	fmt.Println(os.Args)

	// Convert SSH remote url to HTTPS
	url := getRemoteUrl()
	fmt.Println("url", url)

	// Is the tag already?
	tagName, err := checkTag(os.Args[1])
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("tag", tagName)

	if len(tagName) == 1 {
		fmt.Println("Tag name too short")
		os.Exit(1)
	}

	// Ask for a tag message type?
	var tagMessage string
	askedMessage, err := askMessageType()
	if err != nil {
		log.Fatal(err)
	}

	if askedMessage == "Compare tags (default)" {
		recentTag := getRecentTag()
		fmt.Println("recentTag", recentTag)

		tagMessage = url + "/compare/" + recentTag + "..." + tagName
	} else {
		tagMessage = askedMessage
	}

	fmt.Println("Message:", tagMessage)

	// Create the Git tag
	createTag(tagName, tagMessage)

	// Push the tag to the remote
	//pushTag(tagName, url)
}

func getRemoteUrl() string {
	cmd := exec.Command("git", "config", "--get", "remote.origin.url")
	out, err := cmd.Output()

	if err != nil {
		log.Fatal("Error getting remote URL", err)
	}

	remoteOriginUrl := strings.TrimSpace(string(out))

	// TODO: Is the URL ssh or http?

	//vcsType := strings.Split(remoteOriginUrl, "@")[0]
	originUrl := strings.Split(remoteOriginUrl, "@")[1]

	hostname := strings.Split(originUrl, ":")[0]
	pathname := strings.Split(strings.Split(originUrl, ":")[1], ".")[0]

	return "https://" + hostname + "/" + pathname
}

func createTag(tagName, tagMessage string) {
	cmd := exec.Command("git", "tag", "-a", tagName, "-m", tagMessage)
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	if err := cmd.Run(); err != nil {
		log.Fatalf("Error creating tag: %v", err)
	}
	fmt.Printf("Tag %s created.\n", tagName)
}

func pushTag(tagName, remoteURL string) {
	//pushTagToRemotePrompt := promptui.Select{
	//	Label: "Push tag remote",
	//	Items: []string{"yes", "no"},
	//}
	//
	//_, pushTagToRemoteResult, err := pushTagToRemotePrompt.Run()
	//if err != nil {
	//	log.Fatalf("Push tag to remote prompt failed %v\n", err)
	//}

	//if pushTagToRemoteResult == 'yes' {
	cmd := exec.Command("git", "push", "origin", tagName)
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	if err := cmd.Run(); err != nil {
		log.Fatalf("Error pushing tag to remote: %v", err)
	}
	fmt.Printf("Tag %s pushed to remote %s.\n", tagName, remoteURL)
	//}
}

func checkTag(tagName string) (string, error) {
	// get last tag
	//	git for-each-ref --sort=creatordate --format '%(refname:short)' --count=1
	cmd := exec.Command("git", "describe", "--abbrev=0", "--tags", tagName)
	isTagExists, _ := cmd.Output()

	tag := strings.TrimSpace(string(isTagExists))

	if tag == "" {
		return tagName, nil
	} else {
		return "", errors.New("error: tag already exists")
	}
}

func askMessageType() (string, error) {
	/**
	Q1: Ask for a tag message type
	*/
	messageTypePrompt := promptui.Select{
		Label: "Select message type",
		Items: []string{"Compare tags (default)", "Custom message"},
	}

	_, messageTypeResult, err := messageTypePrompt.Run()
	if err != nil {
		return "", fmt.Errorf("Select message type prompt failed %v\n", err)
	}

	/**
	Q2: Prompt for a custom tag message
	*/
	if messageTypeResult == "Custom message" {
		customMessagePrompt := promptui.Prompt{
			Label:   "Tag message",
			Default: "",
		}

		customMessageResult, err := customMessagePrompt.Run()
		if err != nil {
			return "", fmt.Errorf("Enter custom message prompt failed %v\n", err)
		}

		return customMessageResult, nil
	} else {
		return messageTypeResult, nil
	}
}

func getRecentTag() string {
	// NOTE: If it is a new repository, should it get the oldest commit?
	cmd := exec.Command("git", "describe", "--abbrev=0")
	result, err := cmd.Output()

	if err != nil {
		log.Fatal(err)
	}

	return strings.TrimSpace(string(result))
}
