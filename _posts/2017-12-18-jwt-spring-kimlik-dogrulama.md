---
layout: post
title: JWT Token Spring Boot Api üzerinde kimlik doğrulama!
tags: [spring, jwt, security, rest, authentication, authorization, kimlik dogrulama]
bigimg: https://media.licdn.com/mpr/mpr/AAEAAQAAAAAAAAq-AAAAJDEyOWE0OWFlLWJkOTctNDc5ZC04YzEzLTUxMDcwOTNkZTJiZg.jpg

---

Microservices uygulama geliştirmede yaygın olarak kullanılan **JWT(JSON Web Token)** ile **Spring Boot** kullanarak yapmış olduğum kimlik doğrulama ve Rest servis yetkinlendirme örneğini anlatacağım.

CI Status | Repo 
--------- | ----
[![Build Status](https://travis-ci.org/sisa/spring-security-with-jwt.svg?branch=master)](https://travis-ci.org/sisa) | [![Repo](https://sisa.github.io//img/GitHub-Mark-32px.png)](https://github.com/sisa/spring-security-with-jwt)

## Gereksinimler    

   + Maven 3 
   + JDK 1.8    

Spring Boot Securtiy için **WebSecurityConfigurerAdapter kullanıyoruz.**

~~~
package io.sisa.core.security.conf;

import io.sisa.core.security.AuthenticationTokenFilter;
import io.sisa.core.security.RestAuthenticationEntryPoint;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.authentication.www.BasicAuthenticationFilter;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

/**
 * @author isaozturk
 */

@Configuration
@EnableWebSecurity
@EnableConfigurationProperties(value = {JwtProperties.class})
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurity extends WebSecurityConfigurerAdapter {

	private final UserDetailsService userDetailsService;
	private final BCryptPasswordEncoder bCryptPasswordEncoder;
	private final RestAuthenticationEntryPoint restAuthenticationEntryPoint;

	@Autowired
	public WebSecurity(UserDetailsService userDetailsService, BCryptPasswordEncoder bCryptPasswordEncoder, RestAuthenticationEntryPoint restAuthenticationEntryPoint) {
		this.userDetailsService = userDetailsService;
		this.bCryptPasswordEncoder = bCryptPasswordEncoder;
		this.restAuthenticationEntryPoint = restAuthenticationEntryPoint;
	}

	@Bean
	public AuthenticationTokenFilter authenticationTokenFilterBean() throws Exception {
		return new AuthenticationTokenFilter(authenticationManager());
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.csrf().disable()
				.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
				.and()
				.exceptionHandling().authenticationEntryPoint(restAuthenticationEntryPoint)
				.and()
				.authorizeRequests()
				.antMatchers(HttpMethod.POST, "/auth/login").permitAll()
				.antMatchers("/cities/*").hasRole("ADMIN")
				.antMatchers("/cities").hasRole("STANDARD")
				.anyRequest().authenticated()
				.and()
				.addFilterBefore(authenticationTokenFilterBean(), BasicAuthenticationFilter.class);

	}

	@Override
	public void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder);
	}

	@Bean
	CorsConfigurationSource corsConfigurationSource() {
		final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		source.registerCorsConfiguration("/**", new CorsConfiguration().applyPermitDefaultValues());
		return source;
	}
}
~~~


