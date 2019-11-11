# 插件
## 下载私服的库

	<repositories>
		<repository>
			<id>私服id</id>
			<url>私服地址</url>
			<snapshots>
				<enabled>true</enabled>
				<checksumPolicy>warn</checksumPolicy>
				<updatePolicy>always</updatePolicy>
			</snapshots>
			<releases>
				<enabled>true</enabled>
				<checksumPolicy>fail</checksumPolicy>
				<updatePolicy>never</updatePolicy>
			</releases>
		</repository>
	</repositories>

## maven release
- 命令

        release:clean
        release:prepare
        release:perform
        release:rollback

- 上传配置

        <distributionManagement>
            <snapshotRepository>
                <id>私服id</id>
                <url>私服地址</url>
            </snapshotRepository>
            <repository>
                <id>私服id</id>
                <url>私服地址</url>
            </repository>
        </distributionManagement>

- 项目配置

        <scm>
            <url>项目地址(无.git)</url>
            <connection>scm:git:项目地址(带.git)</connection>
            <developerConnection>scm:git:项目地址(带.git)</developerConnection>
            <tag>HEAD</tag>
        </scm>

- 插件配置

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-release-plugin</artifactId>
            <version>2.5.3</version>
            <configuration>
                <!-- 此处使用@{}引用变量，不能用$ -->
                <tagNameFormat>v@{project.version}</tagNameFormat>
                <arguments>-Dmaven.test.skip=true -Dmaven.javadoc.skip=true</arguments>
                <!-- 子模块跟随父模块版本号 -->
                <autoVersionSubmodules>true</autoVersionSubmodules>
            </configuration>
        </plugin>
        <!-- 此插件解决release后重复上传源文件问题 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.4</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
