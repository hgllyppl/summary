# spring-cloud-open-feign
<span style="color:red">红字</span>为主要执行路径，<span style="color:blue">蓝字</span>为暂不分析的代码

## 架构
spring-cloud-open-feign 由两部分组成，分为 feign 与 spring-cloud-open-feign。
- feign 通过解析预定义的接口来构建代理实现，代理完成对请求数据的序列化以及远程服务调用。
- spring-cloud-open-feign 为每个预定义接口提供了相同的 BeanDefinition(FeignClientFactoryBean) 以及一个 applicationContext 来存储每个接口的配置，并由 FeignContext 来管理。
- 关键配置
	- FeignClientsRegistrar/FeignClientFactoryBean
	- FeignAutoConfiguration/FeignContext/FeignClientsConfiguration
	- FeignRibbonClientAutoConfiguration/DefaultFeignLoadBalancedConfiguration
## 注册 BeanDefinition

	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.TYPE)
	@Documented
	@Import(FeignClientsRegistrar.class)
	public @interface EnableFeignClients

由 @EnableFeignClients 上的 FeignClientsRegistrar 开启对所有 feign 接口的注册。以下为主流程代码

	// 注册缺省的配置与 feignclient
	@Override
	public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
		registerDefaultConfiguration(metadata, registry);
		registerFeignClients(metadata, registry);
	}
	
	// 注册缺省的配置
	private void registerDefaultConfiguration(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
		Map<String, Object> defaultAttrs = metadata.getAnnotationAttributes(EnableFeignClients.class.getName(), true);
		if (defaultAttrs != null && defaultAttrs.containsKey("defaultConfiguration")) {
			String name;
			if (metadata.hasEnclosingClass()) {
				name = "default." + metadata.getEnclosingClassName();
			}
			else {
				name = "default." + metadata.getClassName();
			}
			/**
			 * 注册缺省的配置
			 * @see EnableFeignClients#defaultConfiguration()
			 */
			registerClientConfiguration(registry, name, defaultAttrs.get("defaultConfiguration"));
		}
	}
	
	// 注册 feignclient
	public void registerFeignClients(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
		... 以上部分代码为确定 feignclient 的扫描路径，忽略
		// 遍历 basePackages 并扫描带 @FeignClient 的类
		// 遍历 basePackages 并扫描带 @FeignClient 的类
		for (String basePackage : basePackages) {
			Set<BeanDefinition> candidateComponents = scanner.findCandidateComponents(basePackage);
			// 遍历并注册 FeignClient
			for (BeanDefinition candidateComponent : candidateComponents) {
				if (candidateComponent instanceof AnnotatedBeanDefinition) {
					AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
					AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
					Assert.isTrue(annotationMetadata.isInterface(), "@FeignClient can only be specified on an interface");
					Map<String, Object> attributes = annotationMetadata.getAnnotationAttributes(FeignClient.class.getCanonicalName());
					String name = getClientName(attributes);
					/**
					 * 为每个 feignclient 注册配置
					 * @see FeignClient#configuration() 
					 */
					registerClientConfiguration(registry, name, attributes.get("configuration"));
					// 注册 feignclient
					registerFeignClient(registry, annotationMetadata, attributes);
				}
			}
		}
	}
	
	// 注册 feignclient
	private void registerFeignClient(BeanDefinitionRegistry registry, AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
		String className = annotationMetadata.getClassName();
		// 以 FeignClientFactoryBean 作为 feignclient 的 BeanDefinition
		BeanDefinitionBuilder definition = BeanDefinitionBuilder.genericBeanDefinition(FeignClientFactoryBean.class);
		validate(attributes);
		// 取每个 feignclient 的属性，以便初始化的时候注入
		definition.addPropertyValue("url", getUrl(attributes));
		definition.addPropertyValue("path", getPath(attributes));
		String name = getName(attributes);
		definition.addPropertyValue("name", name);
		String contextId = getContextId(attributes);
		definition.addPropertyValue("contextId", contextId);
		definition.addPropertyValue("type", className);
		definition.addPropertyValue("decode404", attributes.get("decode404"));
		definition.addPropertyValue("fallback", attributes.get("fallback"));
		definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
		definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);

		String alias = contextId + "FeignClient";
		AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();

		boolean primary = (Boolean) attributes.get("primary"); // has a default, won't be
		beanDefinition.setPrimary(primary);

		String qualifier = getQualifier(attributes);
		if (StringUtils.hasText(qualifier)) {
			alias = qualifier;
		}

		// 注册 feignclient
		BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className, new String[] { alias });
		BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
	}
	
 - 注意 @EnableFeignClients、@FeignClient 两个注解上提供的配置，这些配置将参与 feignclient 的初始化
 - 注意 为每个接口提供的 BeanDefinition 都是 FeignClientFactoryBean

## 初始化
当 feignclient 被初始化时，FeignClientFactoryBean.getTarget() 被触发

	<T> T getTarget() {

		/**
		 * @see FeignAutoConfiguration#feignContext()
		 */
		FeignContext context = this.applicationContext.getBean(FeignContext.class);
		Feign.Builder builder = feign(context);
		// 如果 url 为空则使用 name 作为服务访问路径
		if (!StringUtils.hasText(this.url)) {
			if (!this.name.startsWith("http")) {
				this.url = "http://" + this.name;
			}
			else {
				this.url = this.name;
			}
			this.url += cleanPath();
			// 构建带负载均衡的 feignclient 实例
			return (T) loadBalance(builder, context, new HardCodedTarget<>(this.type, this.name, this.url));
		}
	}

	/**
	 * 构建 Feign.Builder
	 * @see FeignClientsConfiguration
	 */
	protected Feign.Builder feign(FeignContext context) {
		// 构建 feignclient 上下文并构建 Feign.Builder
		FeignLoggerFactory loggerFactory = get(context, FeignLoggerFactory.class);
		Logger logger = loggerFactory.create(this.type);

		Feign.Builder builder = get(context, Feign.Builder.class)
				// required values
				.logger(logger)
				.encoder(get(context, Encoder.class))
				.decoder(get(context, Decoder.class))
				.contract(get(context, Contract.class));

		// 继续配置 Feign.Builder
		configureFeign(context, builder);

		return builder;
	}
	
	/**
	 * 构建带负载均衡的 feignclient 实例
	 * @see FeignRibbonClientAutoConfiguration
	 * @see RibbonAutoConfiguration
	 * @see org.springframework.cloud.openfeign.ribbon.DefaultFeignLoadBalancedConfiguration
	 */
	protected <T> T loadBalance(Feign.Builder builder, FeignContext context, HardCodedTarget<T> target) {
		Client client = getOptional(context, Client.class);
		if (client != null) {
			builder.client(client);
			Targeter targeter = get(context, Targeter.class);
			/**
			 * 构建实例
			 * @see Feign.Builder#target(Target)
			 * @see HystrixFeign.Builder#target(Target)
			 */
			return targeter.target(this, builder, context, target);
		}

		throw new IllegalStateException(
				"No Feign Client for loadBalancing defined. Did you forget to include spring-cloud-starter-netflix-ribbon?");
	}

### feignclient 上下文的初始化

	// 从上下文管理器(FeignContext)获取实例
	protected <T> T get(FeignContext context, Class<T> type) {
		T instance = context.getInstance(this.contextId, type);
		if (instance == null) {
			throw new IllegalStateException(
					"No bean found of type " + type + " for " + this.contextId);
		}
		return instance;
	}
	
	// 从上下文中获取对象，如果上下文不存在则创建一个
	public <T> T getInstance(String name, Class<T> type) {
		AnnotationConfigApplicationContext context = getContext(name);
		if (BeanFactoryUtils.beanNamesForTypeIncludingAncestors(context, type).length > 0) {
			return context.getBean(type);
		}
		return null;
	}
	
	// 获取上下文，没有则创建一个
	protected AnnotationConfigApplicationContext getContext(String name) {
		if (!this.contexts.containsKey(name)) {
			synchronized (this.contexts) {
				if (!this.contexts.containsKey(name)) {
					this.contexts.put(name, createContext(name));
				}
			}
		}
		return this.contexts.get(name);
	}
	
	// 创建上下文
	protected AnnotationConfigApplicationContext createContext(String name) {
		// 读取配置并注册到上下文
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
		if (this.configurations.containsKey(name)) {
			for (Class<?> configuration : this.configurations.get(name).getConfiguration()) {
				context.register(configuration);
			}
		}
		// 读取默认配置并注册到上下文
		for (Map.Entry<String, C> entry : this.configurations.entrySet()) {
			if (entry.getKey().startsWith("default.")) {
				for (Class<?> configuration : entry.getValue().getConfiguration()) {
					context.register(configuration);
				}
			}
		}
		// 注册默认配置到上下文
		context.register(PropertyPlaceholderAutoConfiguration.class, this.defaultConfigType);
		// 注册属性
		context.getEnvironment().getPropertySources().addFirst(new MapPropertySource(this.propertySourceName, Collections.<String, Object>singletonMap(this.propertyName, name)));
		// 设置父上下文
		if (this.parent != null) {
			// Uses Environment from parent as well as beans
			context.setParent(this.parent);
			// jdk11 issue
			// https://github.com/spring-cloud/spring-cloud-netflix/issues/3101
			context.setClassLoader(this.parent.getClassLoader());
		}
		// 刷新上下文
		context.setDisplayName(generateDisplayName(name));
		context.refresh();
		return context;
	}

### feignclient 初始化
targeter.target

	@Override
	public <T> T target(FeignClientFactoryBean factory, Feign.Builder feign,
			FeignContext context, Target.HardCodedTarget<T> target) {
		return feign.target(target);
	}
	
	/**
	 * 创建实例
	 * @see ReflectiveFeign#newInstance(Target)
	 */
	public <T> T target(Target<T> target) {
		return build().newInstance(target);
	}
	
	// 创建 feign
	public Feign build() {
		Factory synchronousMethodHandlerFactory = new Factory(client, retryer, requestInterceptors, logger, logLevel, decode404, closeAfterDecode, propagationPolicy);
		ParseHandlersByName handlersByName = new ParseHandlersByName(contract, options, encoder, decoder, queryMapEncoder, errorDecoder, synchronousMethodHandlerFactory);
		return new ReflectiveFeign(handlersByName, invocationHandlerFactory, queryMapEncoder);
	}
	
    /**
     * 创建实例
     */
    @SuppressWarnings("unchecked")
    @Override
    public <T> T newInstance(Target<T> target) {
        // 将目标类型的函数构建成以 函数名字为key、MethodHandler 为value的 map
        Map<String, MethodHandler> nameToHandler = targetToHandlersByName.apply(target);
        Map<Method, MethodHandler> methodToHandler = new LinkedHashMap<Method, MethodHandler>();
        List<DefaultMethodHandler> defaultMethodHandlers = new LinkedList<DefaultMethodHandler>();
        // 遍历目标类型的函数并将其转成以 Method 为key，MethodHandler 为value的 map
        for (Method method : target.type().getMethods()) {
            if (method.getDeclaringClass() == Object.class) {
                continue;
            } else if (Util.isDefault(method)) {
                DefaultMethodHandler handler = new DefaultMethodHandler(method);
                defaultMethodHandlers.add(handler);
                methodToHandler.put(method, handler);
            } else {
                methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));
            }
        }
        /**
         * 创建目标类型的代理实例
         * @see InvocationHandlerFactory.Default#create(Target, Map)
         */
        InvocationHandler handler = factory.create(target, methodToHandler);
        T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(), new Class<?>[]{target.type()}, handler);
        // 绑定缺省函数到代理实例
        for (DefaultMethodHandler defaultMethodHandler : defaultMethodHandlers) {
            defaultMethodHandler.bindTo(proxy);
        }
        return proxy;
    }
## 执行
FeignInvocationHandler.invoke

	// 分发并调用相应的函数
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		if ("equals".equals(method.getName())) {
			try {
				Object otherHandler = args.length > 0 && args[0] != null ? Proxy.getInvocationHandler(args[0]) : null;
				return equals(otherHandler);
			} catch (IllegalArgumentException e) {
				return false;
			}
		} else if ("hashCode".equals(method.getName())) {
			return hashCode();
		} else if ("toString".equals(method.getName())) {
			return toString();
		}
		/**
		 * 分发并调用相应的函数
		 * @see SynchronousMethodHandler#invoke(Object[])
		 */
		return dispatch.get(method).invoke(args);
	}
	
    // 代理方法调用
    @Override
    public Object invoke(Object[] argv) throws Throwable {
        /**
         * 创建 RequestTemplate
         * @see ReflectiveFeign.BuildTemplateByResolvingArgs#create(Object[])
         */
        RequestTemplate template = buildTemplateFromArgs.create(argv);
        Retryer retryer = this.retryer.clone();
        while (true) {
            try {
                // 执行请求并处理结果
                return executeAndDecode(template);
            } catch (RetryableException e) {
                /**
                 * 重试逻辑
                 * @see Retryer.Default#continueOrPropagate(RetryableException)
                 * @see retryer.NEVER_RETRY
                 */
                try {
                    retryer.continueOrPropagate(e);
                } catch (RetryableException th) {
                    Throwable cause = th.getCause();
                    if (propagationPolicy == UNWRAP && cause != null) {
                        throw cause;
                    } else {
                        throw th;
                    }
                }
                if (logLevel != Logger.Level.NONE) {
                    logger.logRetry(metadata.configKey(), logLevel);
                }
                continue;
            }
        }
    }
	
	// 执行请求并处理结果
    Object executeAndDecode(RequestTemplate template) throws Throwable {
        // 创建 request
        Request request = targetRequest(template);

        Response response;
        long start = System.nanoTime();
        try {
            /**
             * 执行请求
             * @see org.springframework.cloud.openfeign.ribbon.LoadBalancerFeignClient#execute
             */
            response = client.execute(request, options);
        } catch (IOException e) {
            ...
        }

        boolean shouldClose = true;
        try {
            if (Response.class == metadata.returnType()) {
                if (response.body() == null) {
                    return response;
                }
                if (response.body().length() == null ||
                        response.body().length() > MAX_RESPONSE_BUFFER_SIZE) {
                    shouldClose = false;
                    return response;
                }
                // Ensure the response body is disconnected
                byte[] bodyData = Util.toByteArray(response.body().asInputStream());
                return response.toBuilder().body(bodyData).build();
            }
            // 处理结果
            if (response.status() >= 200 && response.status() < 300) {
                if (void.class == metadata.returnType()) {
                    return null;
                } else {
                    Object result = decode(response);
                    shouldClose = closeAfterDecode;
                    return result;
                }
            } else if (decode404 && response.status() == 404 && void.class != metadata.returnType()) {
                // 处理 404
                Object result = decode(response);
                shouldClose = closeAfterDecode;
                return result;
            } else {
                throw errorDecoder.decode(metadata.configKey(), response);
            }
        } catch (IOException e) {
            ...
        } finally {
            ...
        }
    }
	
	// 执行请求
	@Override
	public Response execute(Request request, Request.Options options) throws IOException {
		try {
			// 将 host 与 url 分离，host 作为 ribbon-client 的上下文名称
			URI asUri = URI.create(request.url());
			String clientName = asUri.getHost();
			URI uriWithoutHost = cleanUrl(request.url(), clientName);
			// 构建 RibbonRequest
			RibbonRequest ribbonRequest = new RibbonRequest(this.delegate, request, uriWithoutHost);
			// 获取 ribbon-client 配置
			// 注意 feign 配置与 ribbon 配置互斥，优先使用 ribbon 配置
			IClientConfig requestConfig = getClientConfig(options, clientName);
			// 执行请求
			RibbonResponse ribbonResponse = lbClient(clientName).executeWithLoadBalancer(ribbonRequest, requestConfig);
			// 返回结果
			return ribbonResponse.toResponse();
		}
		catch (ClientException e) {
			IOException io = findIOException(e);
			if (io != null) {
				throw io;
			}
			throw new RuntimeException(e);
		}
	}
	
	public T executeWithLoadBalancer(final S request, final IClientConfig requestConfig) throws ClientException {
        LoadBalancerCommand<T> command = buildLoadBalancerCommand(request, requestConfig);

        try {
            return command.submit(
                new ServerOperation<T>() {
                    @Override
                    public Observable<T> call(Server server) {
                        // 构造完整的url
                        URI finalUri = reconstructURIWithServer(server, request.getUri());
                        S requestForServer = (S) request.replaceUri(finalUri);
                        try {
                            /**
                             * 真·执行请求
                             * @see org.springframework.cloud.openfeign.ribbon.FeignLoadBalancer#execute
                             */
                            return Observable.just(AbstractLoadBalancerAwareClient.this.execute(requestForServer, requestConfig));
                        }
                        catch (Exception e) {
                            return Observable.error(e);
                        }
                    }
                })
                .toBlocking()
                .single();
        } catch (Exception e) {
            Throwable t = e.getCause();
            if (t instanceof ClientException) {
                throw (ClientException) t;
            } else {
                throw new ClientException(e);
            }
        }

    }
	
	@Override
	public RibbonResponse execute(RibbonRequest request, IClientConfig configOverride)
			throws IOException {
		Request.Options options;
		// 设置超时时间
		if (configOverride != null) {
			RibbonProperties override = RibbonProperties.from(configOverride);
			options = new Request.Options(override.connectTimeout(this.connectTimeout), override.readTimeout(this.readTimeout));
		}
		else {
			options = new Request.Options(this.connectTimeout, this.readTimeout);
		}
		/**
		 * 使用 client 执行请求
		 * @see LoadBalancerFeignClient#execute(Request, Request.Options)
		 * @see DefaultFeignLoadBalancedConfiguration
		 */
		Response response = request.client().execute(request.toRequest(), options);
		return new RibbonResponse(request.getUri(), response);
	}
	
	// 默认使用 HttpURLConnection 执行请求
	@Override
	public Response execute(Request request, Options options) throws IOException {
		HttpURLConnection connection = convertAndSend(request, options);
		return convertResponse(connection, request);
	}
	
	HttpURLConnection convertAndSend(Request request, Options options) throws IOException {
		// 创建 HttpURLConnection
		final HttpURLConnection connection = (HttpURLConnection) new URL(request.url()).openConnection();
		if (connection instanceof HttpsURLConnection) {
			HttpsURLConnection sslCon = (HttpsURLConnection) connection;
			if (sslContextFactory != null) {
				sslCon.setSSLSocketFactory(sslContextFactory);
			}
			if (hostnameVerifier != null) {
				sslCon.setHostnameVerifier(hostnameVerifier);
			}
		}
		// 设置超时时间
		connection.setConnectTimeout(options.connectTimeoutMillis());
		connection.setReadTimeout(options.readTimeoutMillis());
		connection.setAllowUserInteraction(false);
		connection.setInstanceFollowRedirects(options.isFollowRedirects());
		// 设置请求方法
		connection.setRequestMethod(request.httpMethod().name());
		// 是否压缩
		Collection<String> contentEncodingValues = request.headers().get(CONTENT_ENCODING);
		boolean gzipEncodedRequest = contentEncodingValues != null && contentEncodingValues.contains(ENCODING_GZIP);
		boolean deflateEncodedRequest = contentEncodingValues != null && contentEncodingValues.contains(ENCODING_DEFLATE);
		// 设置 header
		boolean hasAcceptHeader = false;
		Integer contentLength = null;
		for (String field : request.headers().keySet()) {
			if (field.equalsIgnoreCase("Accept")) {
				hasAcceptHeader = true;
			}
			for (String value : request.headers().get(field)) {
				if (field.equals(CONTENT_LENGTH)) {
					if (!gzipEncodedRequest && !deflateEncodedRequest) {
						contentLength = Integer.valueOf(value);
						connection.addRequestProperty(field, value);
					}
				} else {
					connection.addRequestProperty(field, value);
				}
			}
		}
		// 添加默认的 accept
		if (!hasAcceptHeader) {
			connection.addRequestProperty("Accept", "*/*");
		}
		// 带压缩的输出 body
		if (request.requestBody().asBytes() != null) {
			if (contentLength != null) {
				connection.setFixedLengthStreamingMode(contentLength);
			} else {
				connection.setChunkedStreamingMode(8196);
			}
			connection.setDoOutput(true);
			OutputStream out = connection.getOutputStream();
			if (gzipEncodedRequest) {
				out = new GZIPOutputStream(out);
			} else if (deflateEncodedRequest) {
				out = new DeflaterOutputStream(out);
			}
			try {
				out.write(request.requestBody().asBytes());
			} finally {
				try {
					out.close();
				} catch (IOException suppressed) { // NOPMD
				}
			}
		}
		return connection;
	}
	
	Response convertResponse(HttpURLConnection connection, Request request) throws IOException {
		// 获取响应信息
		int status = connection.getResponseCode();
		String reason = connection.getResponseMessage();

		if (status < 0) {
			throw new IOException(format("Invalid status(%s) executing %s %s", status, connection.getRequestMethod(), connection.getURL()));
		}
		// 获取 header
		Map<String, Collection<String>> headers = new LinkedHashMap<String, Collection<String>>();
		for (Map.Entry<String, List<String>> field : connection.getHeaderFields().entrySet()) {
			// response message
			if (field.getKey() != null) {
				headers.put(field.getKey(), field.getValue());
			}
		}
		// 获取响应长度
		Integer length = connection.getContentLength();
		if (length == -1) {
			length = null;
		}
		InputStream stream;
		if (status >= 400) {
			stream = connection.getErrorStream();
		} else {
			stream = connection.getInputStream();
		}
		// 构造 Response
		return Response.builder()
				.status(status)
				.reason(reason)
				.headers(headers)
				.request(request)
				.body(stream, length)
				.build();
	}