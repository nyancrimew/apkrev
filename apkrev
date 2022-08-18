#!/bin/sh

PACKAGE="$1"
WORK_DIRECTORY="$HOME/reversing"
APK_LOCATION="$WORK_DIRECTORY/apks/$PACKAGE.apk"
OUTPUT_DIRECTORY="$WORK_DIRECTORY/reversing/$PACKAGE"
COOKIE_JAR="cookies.txt"

echo "  grabbing download url from apkpure..."
PAGE_CONTENT="$(curl_chrome101 -s -L --cookie-jar $COOKIE_JAR https://apkpure.com/dl/$PACKAGE/download?from=details)"
DOWNLOAD_URL="$(htmlq --text --attribute href 'a#download_link' <<< "$PAGE_CONTENT")"
TYPE_TAG="$(htmlq --text 'div.info-title span' <<< "$PAGE_CONTENT")"

if [ "$DOWNLOAD_URL" == "" ]; then
  echo "    couldn't find '$PACKAGE' on apkpure, aborting"
  exit 1
fi

echo "  downloading apk..."
curl_chrome101 --progress-bar -L --cookie-jar $COOKIE_JAR --output "$APK_LOCATION" "$DOWNLOAD_URL"

if [ ! -f "$APK_LOCATION" ]; then
  echo "    couldn't download apk, aborting"
  exit 1
fi

if [ "$TYPE_TAG" = "XAPK" ]; then
  echo "    unpacking xapk..."
  unzip -o -q "$APK_LOCATION" "*.apk" -d "$APK_LOCATION.out"
  if [ $? -ne 0 ]; then
    echo "      couldn't unpack xapk, aborting"
    exit 1
  fi
  APK_LOCATION="$APK_LOCATION.out/*.apk"
fi

echo "  decompiling apk..."

echo "     running jadx..."
jadx -d "$OUTPUT_DIRECTORY" --deobf --deobf-parse-kotlin-metadata --deobf-use-sourcename --use-dx --show-bad-code $APK_LOCATION

if [ $? -ne 0 ]; then 
  echo "      jadx failed, aborting"
  exit 1
fi

RN_BUNDLE="$OUTPUT_DIRECTORY/resources/assets/index.android.bundle"

if [ -f "$RN_BUNDLE" ]; then
  echo "        react native bundle found, attempting to unpack..."
  
  FILE_TYPE="$(file -b $RN_BUNDLE)"
  if grep -q "Hermes" <<< "$FILE_TYPE"; then
    echo "         running hbctool..."
    hbctool disasm "$RN_BUNDLE" "$OUTPUT_DIRECTORY/hasm"
    if [ $? -ne 0 ]; then
      echo "           hbctool failed, aborting"
      exit 1
    fi
  else
    echo "        running react-native-decompiler..."
    npx react-native-decompiler -i "$RN_BUNDLE" -o "$OUTPUT_DIRECTORY/rn-sources" && rm "$OUTPUT_DIRECTORY/rn-sources/null.cache"
    if [ $? -ne 0 ]; then
      echo "           react-native-decompiler failed, aborting"
      exit 1
    fi
  fi
fi

echo "  opening in codium..."
codium "$OUTPUT_DIRECTORY"