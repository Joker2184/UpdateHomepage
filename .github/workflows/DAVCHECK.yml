name: Test Alist WebDAV Connection

on:
  workflow_dispatch:  # 允许手动触发

jobs:
  test_webdav_connection:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository (Optional)
        uses: actions/checkout@v4
        # 如果不需要仓库代码可删除此步骤

      - name: Install rclone
        run: sudo curl https://rclone.org/install.sh | sudo bash

      - name: Configure rclone for WebDAV
        run: |
          rclone config create alist_test webdav \
            url "http://${{ secrets.WEBDAV_HOST }}:${{ secrets.WEBDAV_PORT }}/dav" \
            user "${{ secrets.ALIST_USERNAME }}" \
            pass "${{ secrets.ALIST_PASSWORD }}"

      - name: Test Basic Connectivity (Ping)
        run: |
          # 提取域名/IP（去除http://和端口）
          HOST=$(echo "${{ secrets.WEBDAV_HOST }}" | sed 's~http://~~;s~https://~~;s~:.*~~')
          echo "Testing connectivity to $HOST ..."
          ping -c 4 $HOST || echo "Ping failed (可能禁用了ICMP)"

      - name: Test WebDAV Port Reachability
        run: |
          HOST=$(echo "${{ secrets.WEBDAV_HOST }}" | sed 's~http://~~;s~https://~~;s~:.*~~')
          PORT="${{ secrets.WEBDAV_PORT }}"
          timeout 5 nc -zv $HOST $PORT || echo "Port $PORT is unreachable"

      - name: List Root Directory (验证基础权限)
        run: |
          echo "尝试列出WebDAV根目录内容："
          rclone ls alist_test:/ --max-depth 1 --log-level DEBUG

      - name: Check Write Permission (可选)
        run: |
          TEST_FILE="github-actions-test-file.txt"
          echo "Testing write permission..." > $TEST_FILE
          rclone copy $TEST_FILE alist_test:/ --log-level DEBUG
          rclone deletefile alist_test:/$TEST_FILE --log-level DEBUG
