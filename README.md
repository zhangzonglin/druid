因业务需要，系统需要定时更新下载文件(.csv)，并更新数据库。由于数据量较大，决定使用“load data infile”语句，本地数据库可以正常执行，但放到spring的定时任务中后，一直报“MySqlLoadDataInFileStatement not allow”的异常。
系统数据库使用的Druid，查询发现应该是wallfilter的问题，详细信息可查看“https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE-wallfilter” ,但并未发现MySqlLoadDataInFileStatement的配置项，后代码跟踪发现errorcode为“1999”，
所以判定为“noneBaseStatementAllow”。随后修改配置文件，配置如下：(ps:filters和proxyFilters的配置是组合关系，而不是替换关系)
  <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <!-- 基本属性 url、user、password -->
        <property name="url" value="${connection.jdbcUrl}"/>
        <property name="username" value="${connection.username}"/>
        <property name="password" value="${connection.password}"/>

        <!-- 配置初始化大小、最小、最大 -->
        <property name="initialSize" value="${druid.initialSize}"/>
        <property name="minIdle" value="${druid.minIdle}"/>
        <property name="maxActive" value="${druid.maxActive}"/>

        <!-- 配置获取连接等待超时的时间 -->
        <property name="maxWait" value="${druid.maxWait}"/>
        <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
        <property name="timeBetweenEvictionRunsMillis" value="${druid.timeBetweenEvictionRunsMillis}"/>

        <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
        <property name="minEvictableIdleTimeMillis" value="${druid.minEvictableIdleTimeMillis}"/>

        <property name="validationQuery" value="${druid.validationQuery}"/>
        <property name="testWhileIdle" value="${druid.testWhileIdle}"/>
        <property name="testOnBorrow" value="${druid.testOnBorrow}"/>
        <property name="testOnReturn" value="${druid.testOnReturn}"/>

        <!-- 打开PSCache，并且指定每个连接上PSCache的大小  如果用Oracle，则把poolPreparedStatements配置为true，mysql可以配置为false。-->
        <property name="poolPreparedStatements" value="${druid.poolPreparedStatements}"/>
        <property name="maxPoolPreparedStatementPerConnectionSize"
                  value="${druid.maxPoolPreparedStatementPerConnectionSize}"/>

        <!-- 配置监控统计拦截的filters -->
        <property name="filters" value="stat"/>
        <property name="proxyFilters">
            <list>
                <ref bean="wall-filter"/>
            </list>
        </property>

    </bean>

    <bean id="wall-filter" class="com.alibaba.druid.wall.WallFilter">
        <property name="config" ref="wall-filter-config" />
    </bean>

    <bean id="wall-filter-config" class="com.alibaba.druid.wall.WallConfig" init-method="init">
        <property name="dir" value="META-INF/druid/wall/mysql" />
        <property name="noneBaseStatementAllow" value="true"/>
    </bean>
#   d r u i d  
 